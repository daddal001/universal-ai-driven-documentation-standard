---
title: "Observability Standard — Logging, Tracing & Metrics"
type: "standard"
status: "approved"
owner: "@platform-team"
created: "2026-03-05"
last_updated: "2026-03-05"
version: "1.0.0"
---

# Observability Standard v1.0.0

Canonical reference for all logging, tracing, and metrics instrumentation in your AI platform.
Every service, function, and network call MUST comply with this standard.

Based on: OpenTelemetry Semantic Conventions v1.40.0, Google SRE Book, OWASP Logging Cheat Sheet,
EU AI Act Articles 12/19/50, Stripe/Netflix/Uber production patterns.

---

## 1. Structured Logging

### 1.1 Every Log Call MUST Use `extra={}`

```python
# CORRECT
logger.info("Document processed", extra={
    "service_name": "backend-document",
    "document_id": doc_id,
    "page_count": pages,
    "duration_ms": round(elapsed * 1000, 2),
})

# WRONG — unstructured, invisible to Loki/LogQL
logger.info("Document %s processed in %.2fs", doc_id, elapsed)
```

### 1.2 Mandatory Fields (EVERY log line)

| Field | Source | Purpose |
|-------|--------|---------|
| `service_name` | Hardcoded per service | Identify origin |
| `trace_id` | Auto-injected by `LoggingInstrumentor` | Trace correlation |
| `span_id` | Auto-injected by `LoggingInstrumentor` | Span correlation |

These three are injected automatically by `opentelemetry-instrumentation-logging`.
Service code must provide `service_name` in `extra={}`.

### 1.3 Recommended Fields (per context)

| Context | Fields |
|---------|--------|
| HTTP requests | `http_method`, `http_path`, `http_status`, `client_ip`, `duration_ms` |
| Authentication | `user_id` (truncated), `auth_method`, `session_prefix` |
| Database ops | `db_operation`, `db_table`, `row_count`, `duration_ms` |
| AI interactions | `model_id`, `input_tokens`, `output_tokens`, `duration_ms` |
| File operations | `filename`, `file_size`, `operation` |
| Errors | `error_type`, `error_message` (first 200 chars) |
| Security events | `security_event`, `violation_type`, `severity` |

### 1.4 NEVER Log (OWASP + GDPR)

- Access tokens, refresh tokens, API keys
- Passwords or password hashes
- Session IDs (use hashed prefix only: `session_id[:8]`)
- Raw PII (names, emails, phone numbers) — use pseudonymized hash
- Database connection strings
- Encryption keys
- Full request/response bodies containing user data

### 1.5 Log Levels

| Level | Use When |
|-------|----------|
| `ERROR` | Operation failed, requires investigation |
| `WARNING` | Degraded operation, security event, approaching limits |
| `INFO` | Significant business event (login, document processed, task completed) |
| `DEBUG` | Diagnostic detail (only in dev, never in prod hot paths) |

---

## 2. Distributed Tracing

### 2.1 Span Naming Conventions

| Operation Type | Pattern | Example |
|---------------|---------|---------|
| HTTP Server | `{METHOD} {route}` | `POST /api/metadata/process` |
| HTTP Client | `{service}.http.{operation}` | `auth.keycloak.http.exchange_code` |
| Database | `{db_system}.{operation}` | `postgresql.SELECT task_documents` |
| Cache | `redis.{operation}` | `redis.GET session` |
| Business Logic | `{component}.{operation}` | `orchestrator.process_batch` |
| Security | `security.{check}` | `security.validate_message` |
| Background Task | `celery.task.{name}` | `celery.task.parse_document` |

**Critical rule:** Span names MUST be low-cardinality. Never include IDs, timestamps, or user data.

### 2.2 Required Span Attributes by Type

#### HTTP Server Spans (FastAPI routes)

```python
span.set_attribute("http.request.method", request.method)
span.set_attribute("url.path", request.url.path)
span.set_attribute("http.route", "/api/metadata/tasks/{task_id}/status")
span.set_attribute("http.response.status_code", response.status_code)
span.set_attribute("client.address", request.client.host)
span.set_attribute("http.request.body.size", len(body))
```

#### HTTP Client Spans (httpx calls to other services)

```python
span.set_attribute("http.request.method", method)
span.set_attribute("server.address", "backend-ai.app.svc")
span.set_attribute("server.port", 8002)
span.set_attribute("url.full", url)  # credentials redacted
span.set_attribute("http.response.status_code", response.status_code)
```

#### Database Spans (PostgreSQL, Redis)

```python
span.set_attribute("db.system.name", "postgresql")
span.set_attribute("db.operation.name", "SELECT")
span.set_attribute("db.namespace", "app_metadata")
span.set_attribute("db.collection.name", "task_documents")
span.set_attribute("server.address", "postgres.app-db.svc")
```

#### Business Logic Spans

```python
span.set_attribute("document.id", doc_id)
span.set_attribute("document.page_count", page_count)
span.set_attribute("user.id", user_id[:8])  # truncated
span.set_attribute("task.id", task_id)
span.set_attribute("operation.result", "success")
span.set_attribute("duration_ms", round(elapsed * 1000, 2))
```

### 2.3 Error Recording (EVERY catch block)

```python
try:
    result = await process()
except Exception as e:
    span.set_attribute("error.type", type(e).__name__)
    span.record_exception(e)
    span.set_status(trace.StatusCode.ERROR, str(e)[:200])
    logger.error("Operation failed", extra={
        "error_type": type(e).__name__,
        "error_message": str(e)[:200],
        "service_name": SERVICE_NAME,
    })
    raise
```

### 2.4 Span Status Rules

| HTTP Status | Server Span Status | Client Span Status |
|-------------|-------------------|-------------------|
| 1xx, 2xx, 3xx | Unset | Unset |
| 4xx | **Unset** (client error) | Error |
| 5xx | Error | Error |
| Network error | Error | Error |

### 2.5 Context Propagation

- W3C `traceparent` header for HTTP (auto-propagated by OTEL)
- Celery: inject context into task headers, extract on worker side
- SSE: include `trace_id` in event metadata for frontend correlation
- Frontend: `FetchInstrumentation` with `propagateTraceHeaderCorsUrls`

---

## 3. OWASP Security Event Logging

Use the standard OWASP Logging Vocabulary (53 event types).
Reference: `services/common/observability/audit.py`

### 3.1 Mandatory Security Events

| Event | When | Severity |
|-------|------|----------|
| `authn_login_success` | Successful authentication | LOW |
| `authn_login_fail` | Failed authentication | MEDIUM |
| `authn_login_lock` | Account locked | HIGH |
| `authn_token_created` | Token issued | LOW |
| `authn_token_revoked` | Token revoked | LOW |
| `authz_fail` | Unauthorized access attempt (IDOR) | HIGH |
| `session_created` | New session | LOW |
| `session_expired` | Session ended | LOW |
| `input_validation_fail` | XSS/SQLi/injection blocked | HIGH |
| `excess_rate_limit_exceeded` | Rate limit hit | MEDIUM |
| `sys_startup` | Service started | LOW |
| `sys_shutdown` | Service stopped | LOW |
| `upload_complete` | File uploaded | LOW |
| `upload_validation` | File scanned | MEDIUM |

### 3.2 Security Event Format

```python
log_owasp_event(
    event_type="authn_login_fail",
    actor=user_id or "unknown",
    resource="keycloak_token_endpoint",
    outcome="failure",
    ip_address=client_ip,
    details={"reason": "invalid_code", "grant_type": "authorization_code"},
)
```

---

## 4. EU AI Act Compliance Logging

### 4.1 Required Fields for AI Interactions (Articles 12, 19)

Every AI model call (Gemini, etc.) MUST log:

| Field | Type | Purpose |
|-------|------|---------|
| `interaction_id` | string | Unique interaction identifier |
| `timestamp_start` | ISO 8601 | Start of AI interaction |
| `timestamp_end` | ISO 8601 | End of AI interaction |
| `user_id_pseudonymized` | string | SHA-256 hash of user_id |
| `model_id` | string | `gemini-2.0-flash` |
| `model_version` | string | Exact model version |
| `model_provider` | string | `google-vertex-ai` |
| `input_token_count` | int | Input token count |
| `output_token_count` | int | Output token count |
| `processing_time_ms` | float | Total duration |
| `content_filter_applied` | bool | Safety filter outcome |
| `transparency_notice_shown` | bool | User was informed of AI |
| `trace_id` | string | OpenTelemetry trace ID |

### 4.2 Retention

- Minimum 6 months for all AI interaction logs (Article 19)
- Store in `audit_trail` PostgreSQL table with tamper-evident hash chain

---

## 5. Frontend Browser Tracing

### 5.1 Required Browser Instrumentations

| Instrumentation | Purpose |
|----------------|---------|
| `FetchInstrumentation` | Traces all `fetch()` API calls |
| `DocumentLoadInstrumentation` | Page load performance |
| `UserInteractionInstrumentation` | Click/submit events |

### 5.2 Required `propagateTraceHeaderCorsUrls`

```typescript
new FetchInstrumentation({
    propagateTraceHeaderCorsUrls: [/\/api\//],
    ignoreUrls: [/\/health/, /\/metrics/],
})
```

### 5.3 Custom Browser Spans

```typescript
const tracer = trace.getTracer('frontend-lite');

// API calls
const span = tracer.startSpan('api.sendMessage', {
    attributes: {
        'http.method': 'POST',
        'http.url': '/api/chat/send',
        'message.length': message.length,
    },
});

// SSE connections
const span = tracer.startSpan('sse.connect', {
    attributes: {
        'sse.url': url,
        'session.id': sessionId?.substring(0, 8),
    },
});
```

---

## 6. Metrics (Four Golden Signals)

Every service MUST expose these Prometheus metrics:

| Signal | Metric | Type |
|--------|--------|------|
| **Latency** | `http_request_duration_seconds` | Histogram |
| **Traffic** | `http_requests_total` | Counter |
| **Errors** | `http_errors_total` | Counter |
| **Saturation** | `process_resident_memory_bytes` | Gauge |

SpanMetrics connector in OTEL Collector auto-generates RED metrics from traces.

---

## 7. Checklist for New Code

- [ ] Every function with I/O has a `tracer.start_as_current_span()` with semantic attributes
- [ ] Every `logger.xxx()` call has `extra={}` with at least `service_name`
- [ ] Every `except` block calls `span.record_exception(e)` and `span.set_status(ERROR)`
- [ ] Every HTTP client call sets `server.address`, `http.request.method`, `http.response.status_code`
- [ ] Every DB call sets `db.system.name`, `db.operation.name`, `db.collection.name`
- [ ] Every security event uses OWASP vocabulary via `log_owasp_event()`
- [ ] Every AI interaction logs EU AI Act fields
- [ ] No PII/tokens/keys in log messages or span attributes
- [ ] Span names are low-cardinality (no IDs or timestamps)

---

**Previous:** [42 - Metrics](./42-METRICS.md)
**Next:** [44 - Architecture](./44-ARCHITECTURE.md)
