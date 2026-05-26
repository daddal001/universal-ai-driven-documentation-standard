---
title: "Security & Compliance Documentation"
type: "standard"
status: "approved"
owner: "@security-team"
classification: "public"
created: "2025-12-09"
last_updated: "2025-12-12"
version: "1.0.0"
---

# Security & Compliance Documentation

> 💡 **Open Source Projects:** This standard is optional for most OSS projects.
> See [38-CONTEXT_GUIDANCE](./38-CONTEXT_GUIDANCE.md) for what you actually need.

> **Goal:** Ensure documentation itself doesn't become a security risk while enabling compliance traceability for audits.

---

## 1. Data Classification in Documentation

### Classification Levels

| Level | Description | Documentation Rules |
|-------|-------------|---------------------|
| **Public** | Can be shared externally | No restrictions |
| **Internal** | Company-wide access | Default for most docs |
| **Confidential** | Need-to-know basis | Restricted access, no screenshots of data |
| **Restricted** | Highly sensitive | Approval required, audit logging |

### Required Frontmatter

```yaml
---
title: "Payment Processing Guide"
classification: "confidential"
data_sensitivity:
  - "PII"
  - "financial"
compliance:
  - "PCI-DSS"
  - "GDPR"
access_control: "finance-team, payment-engineers"
last_security_review: "2025-12-01"
---
```

---

## 2. PII Handling in Documentation

### Never Include

| ❌ Never Document | ✅ Use Instead |
|-------------------|----------------|
| Real customer emails | `user@example.com` |
| Real names | `John Doe`, `Jane Smith` |
| Real phone numbers | `+1-555-123-4567` |
| Real addresses | `123 Example St, Anytown` |
| Real API keys | `sk-XXXXXXXXXXXXXXXX` |
| Real passwords | `<your-password>` |
| Production database IDs | Generated test IDs |

### Screenshot Guidelines

```markdown
## Taking Screenshots for Documentation

1. ALWAYS use staging/development environments
2. NEVER include real user data
3. Blur or redact any potentially sensitive information
4. Use test accounts with obvious fake data
5. Review screenshots before committing

### Redaction Checklist
- [ ] No real email addresses visible
- [ ] No real names visible
- [ ] No API keys or tokens visible
- [ ] No internal URLs that shouldn't be public
- [ ] No production database records
```

---

## 3. Compliance Traceability

### GDPR Documentation Requirements

| Requirement | Documentation |
|-------------|---------------|
| **Data Mapping** | Document what PII is collected, where stored, how long retained |
| **Legal Basis** | Document why each data category is collected |
| **Data Subject Rights** | Document how users exercise rights (access, deletion) |
| **Third-Party Sharing** | Document all data processors and transfers |

```markdown
## GDPR Data Flow Documentation

### Data: User Email Address

| Attribute | Value |
|-----------|-------|
| **Collected At** | Registration form |
| **Legal Basis** | Contract performance |
| **Storage** | PostgreSQL (users.email) |
| **Retention** | Account lifetime + 30 days |
| **Processors** | SendGrid (email delivery) |
| **User Rights** | Editable in Settings, exportable via GDPR request |
```

### SOC2 Documentation Requirements

| Control | Documentation Required |
|---------|----------------------|
| **Access Control** | Document who has access to what |
| **Change Management** | Document all system changes |
| **Incident Response** | Document incident procedures |
| **Vendor Management** | Document third-party security |

### Compliance Matrix

Maintain `docs/compliance/CONTROL_MATRIX.md`:

```markdown
| Control ID | Description | Documentation | Last Reviewed |
|------------|-------------|---------------|---------------|
| AC-1 | Access Control Policy | [ACCESS.md](./policies/ACCESS.md) | 2025-12-01 |
| IR-1 | Incident Response Plan | [INCIDENT.md](./policies/INCIDENT.md) | 2025-11-15 |
| CM-1 | Change Management | [CHANGES.md](./policies/CHANGES.md) | 2025-12-01 |

> See full [Global Compliance Matrix](../compliance/GLOBAL_MATRIX.md)
```

---

## 4. Security-Sensitive Documentation Review

### Additional Review Requirements

Documentation touching security topics MUST have:

1. **Security Review** — Review by security team member
2. **No Secrets** — Automated scan for hardcoded secrets
3. **Access Control** — Verify appropriate access restrictions

### Pre-Commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
        name: Detect hardcoded secrets

  - repo: local
    hooks:
      - id: check-pii
        name: Check for PII patterns
        entry: scripts/check-pii.sh
        language: script
        files: \.(md|txt)$
```

### PII Detection Script

```bash
#!/bin/bash
# scripts/check-pii.sh

# Patterns that suggest real PII
PATTERNS=(
    '[a-zA-Z0-9._%+-]+@(?!example\.com)[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'  # Real emails
    '\b\d{3}-\d{2}-\d{4}\b'  # SSN format
    '\b\d{16}\b'  # Credit card numbers
    'sk-[a-zA-Z0-9]{32,}'  # API keys
)

for pattern in "${PATTERNS[@]}"; do
    if grep -rPn "$pattern" "$@"; then
        echo "❌ Potential PII detected. Please review."
        exit 1
    fi
done

echo "✅ No PII patterns detected"
```

---

## 5. Secure Documentation Practices

### Environment-Specific Documentation

```markdown
## Configuration

### Development
```bash
export API_URL="http://localhost:8000"
export DEBUG=true
```

### Production
>
> ⚠️ Production values are stored in Vault.
> See [Secrets Management](./SECRETS.md) for access.

```

### Credential References

```markdown
## Authentication Setup

1. Get API credentials from [Vault](https://vault.example.com/secrets/api)
2. Store in environment:
   ```bash
   export API_KEY="$VAULT_API_KEY"  # Retrieved from Vault
   ```

> 🔒 Never commit credentials. Use `doppler` or `vault` CLI.

```

---

## 6. Audit Trail

### Documentation Changes

For compliance-sensitive documentation:

```yaml
# Commit message format for compliance docs
git commit -m "docs(compliance): Update data retention policy

- Changed retention from 90 to 30 days per legal review
- Updated GDPR data mapping
- Ticket: COMPLY-123
- Reviewed-by: @security-lead
- Approved-by: @dpo"
```

---

## 7. Authentication Documentation Standards

### OAuth 2.1 Documentation Requirements

OAuth 2.1 consolidates security best practices. Document these mandatory elements:

| Element | Requirement | Documentation |
|---------|-------------|---------------|
| **PKCE** | Required for all clients | Document code_verifier generation |
| **Refresh Token Rotation** | Required | Document rotation behavior |
| **No Implicit Grant** | Deprecated | Note removal from supported flows |
| **Exact Redirect URI Match** | Required | Document registered URIs |

#### OAuth 2.1 Flow Documentation Template

```markdown
## OAuth 2.1 Authentication

### Supported Flows

| Flow | Use Case | PKCE Required |
|------|----------|---------------|
| Authorization Code + PKCE | Web apps, mobile apps | ✅ Yes |
| Client Credentials | Service-to-service | ❌ No |

> ⚠️ **Deprecated:** Implicit grant and Resource Owner Password flow are NOT supported.

### Authorization Code Flow with PKCE

1. Generate `code_verifier` (43-128 chars, URL-safe)
2. Create `code_challenge` = BASE64URL(SHA256(code_verifier))
3. Redirect to:
   ```

   GET /oauth/authorize
     ?response_type=code
     &client_id={client_id}
     &redirect_uri={redirect_uri}
     &scope={scopes}
     &state={random_state}
     &code_challenge={code_challenge}
     &code_challenge_method=S256

   ```
4. Exchange code for tokens:
   ```bash
   POST /oauth/token
   Content-Type: application/x-www-form-urlencoded

   grant_type=authorization_code
   &code={authorization_code}
   &redirect_uri={redirect_uri}
   &client_id={client_id}
   &code_verifier={code_verifier}
   ```

### Token Response

```json
{
  "access_token": "eyJhbG...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
  "scope": "read write"
}
```

```

### JWT Documentation Requirements

Document these JWT elements for all API authentication:

| Element | Must Document | Example |
|---------|--------------|---------|
| **Algorithm** | Signing algorithm used | `RS256`, `ES256` (never HS256 for APIs) |
| **Issuer (iss)** | Token issuer URL | `https://auth.example.com` |
| **Audience (aud)** | Intended recipient | `https://api.example.com` |
| **Expiration** | Token lifetime | `exp`: 1 hour for access, 7 days for refresh |
| **Claims** | Custom claims used | `roles`, `permissions`, `tenant_id` |

#### JWT Structure Documentation

```markdown
## JWT Token Structure

### Access Token Claims

| Claim | Type | Description |
|-------|------|-------------|
| `sub` | string | User ID (UUID) |
| `iss` | string | `https://auth.example.com` |
| `aud` | string | `https://api.example.com` |
| `exp` | number | Expiration (Unix timestamp) |
| `iat` | number | Issued at (Unix timestamp) |
| `scope` | string | Space-delimited scopes |
| `roles` | array | User roles: `["admin", "editor"]` |

### Token Validation Checklist

API consumers MUST validate:
- [ ] Signature (using public key from JWKS endpoint)
- [ ] `exp` claim (token not expired)
- [ ] `iss` claim (matches expected issuer)
- [ ] `aud` claim (includes this API)

### JWKS Endpoint

Public keys for signature verification:
```

GET <https://auth.example.com/.well-known/jwks.json>

```

Response:
```json
{
  "keys": [{
    "kty": "RSA",
    "kid": "key-id-1",
    "use": "sig",
    "alg": "RS256",
    "n": "...",
    "e": "AQAB"
  }]
}
```

```

### Security Best Practices Documentation

```markdown
## Security Requirements

### Token Storage

| Platform | Access Token | Refresh Token |
|----------|--------------|---------------|
| Web (SPA) | Memory only | HttpOnly cookie (Secure, SameSite=Strict) |
| Mobile | Secure enclave/keychain | Secure enclave/keychain |
| Server | Environment variable | Encrypted at rest |

> ⚠️ **Never** store tokens in localStorage or sessionStorage (XSS vulnerable)

### Token Refresh

```typescript
// Automatic token refresh pattern
async function fetchWithAuth(url: string, options: RequestInit) {
  let response = await fetch(url, addAuthHeader(options));

  if (response.status === 401) {
    await refreshAccessToken();
    response = await fetch(url, addAuthHeader(options));
  }

  return response;
}
```

### Logout / Token Revocation

```
POST /oauth/revoke
Content-Type: application/x-www-form-urlencoded

token={refresh_token}
&token_type_hint=refresh_token
```

```

---

## 8. Enterprise Compliance Requirements

### GDPR Documentation Requirements (Detailed)

#### Article 30: Records of Processing Activities

**Required Documentation:**

1. **Data Processing Record Template**
   - See [example-enterprise-compliance.md](./examples/example-enterprise-compliance.md)
   - Must include: purpose, legal basis, data categories, retention, processors
   - Template: `templates/tier-enterprise/DATA_PROCESSING_RECORD.md`

2. **Data Flow Documentation**
   - Document data collection points
   - Document storage locations
   - Document third-party processors
   - Document data transfers (including cross-border)

3. **Legal Basis Documentation**
   - For each data category, document legal basis (consent, contract, legitimate interest, etc.)
   - Document how consent is obtained (if applicable)
   - Document consent withdrawal process

#### Article 33: Breach Notification

**Required Documentation:**

1. **Breach Notification Procedure**
   - 72-hour notification timeline
   - Supervisory authority contact information
   - Breach assessment criteria
   - Notification template

2. **Breach Response Runbook**
   - Detection procedures
   - Containment steps
   - Assessment process
   - Notification decision tree

**Template:** See `templates/tier-enterprise/BREACH_NOTIFICATION.md`

#### Article 35: Data Protection Impact Assessment (DPIA)

**Required Documentation:**

1. **DPIA Template**
   - Description of processing
   - Necessity and proportionality assessment
   - Risk assessment
   - Mitigation measures

2. **DPIA Process**
   - When DPIA is required
   - Who conducts DPIA
   - Review and approval process
   - Documentation requirements

### SOC2 Control Mapping

#### Control: CC6.1 - Logical and Physical Access Controls

**Documentation Requirements:**

| Requirement | Documentation | Evidence Location |
|-------------|---------------|-------------------|
| Access control policy | [Access Control Policy](./docs/ACCESS_CONTROL.md) | Policy document |
| User access reviews | [Access Review Process](./docs/ACCESS_REVIEW.md) | Quarterly review reports |
| Role definitions | [Role Matrix](./docs/ROLES.md) | Role documentation |
| Access logs | [Audit Logs](./docs/AUDIT_LOGS.md) | Log retention records |

**Template:** See `templates/tier-enterprise/COMPLIANCE_MATRIX.md`

#### Control: CC7.2 - System Operations

**Documentation Requirements:**

| Requirement | Documentation | Evidence Location |
|-------------|---------------|-------------------|
| Monitoring procedures | [Monitoring Runbook](./docs/MONITORING.md) | Runbook documentation |
| Alert procedures | [Incident Response Plan](./docs/INCIDENT_RESPONSE.md) | Incident procedures |
| System logs | [Logging Standards](./docs/LOGGING.md) | Logging configuration |

#### Control: CC8.1 - Change Management

**Documentation Requirements:**

| Requirement | Documentation | Evidence Location |
|-------------|---------------|-------------------|
| Change management policy | [Change Management Policy](./docs/CHANGE_MANAGEMENT.md) | Policy document |
| Change approval process | [Change Process](./docs/CHANGE_PROCESS.md) | Process documentation |
| Change logs | [Change Log](./docs/CHANGE_LOG.md) | Change tracking system |

### ISO27001 Documentation Requirements

#### Control: A.9.2.1 - User Registration and De-Registration

**Documentation Requirements:**

- User registration process documented
- User de-registration process documented
- Access provisioning procedures
- Access review schedule

**Template:** See `templates/tier-enterprise/USER_MANAGEMENT.md`

#### Control: A.12.4.1 - Event Logging

**Documentation Requirements:**

- Logging standards and procedures
- Log retention policy
- Log review procedures
- Log storage and protection

**Template:** See `templates/tier-enterprise/LOGGING_STANDARDS.md`

#### Control: A.18.1.1 - Legal Requirements

**Documentation Requirements:**

- Legal and regulatory requirements identified
- Compliance mapping
- Review schedule
- Evidence of compliance

**Template:** See `templates/tier-enterprise/LEGAL_COMPLIANCE.md`

### Audit Trail Documentation Standards

**Required for All Compliance Documentation:**

1. **Change Tracking**
   - All changes to compliance docs must be logged
   - Include: date, author, change type, approval
   - Template: `templates/tier-enterprise/AUDIT_TRAIL.md`

2. **Review Schedule**
   - Quarterly reviews for all compliance documentation
   - Annual reviews for policies
   - Document review dates and outcomes

3. **Approval Process**
   - Technical review by subject matter expert
   - Compliance review by compliance team
   - Final approval by document owner

### Compliance Checklist per Document Type

#### For Data Processing Documentation

- [ ] Data categories clearly defined
- [ ] Legal basis documented for each category
- [ ] Retention periods specified
- [ ] Third-party processors listed
- [ ] Data subject rights documented
- [ ] Security measures described
- [ ] Review date and next review scheduled

#### For Access Control Documentation

- [ ] Access control policy documented
- [ ] Roles and permissions defined
- [ ] Access review process documented
- [ ] Access logs maintained
- [ ] Review schedule established

#### For Incident Response Documentation

- [ ] Incident response plan documented
- [ ] Breach notification procedure included
- [ ] Contact information current
- [ ] Response timeline defined
- [ ] Tested within last 12 months

### Enterprise Governance Models

#### Centralized Model

**Characteristics:**
- Single documentation team
- Centralized review and approval
- Consistent standards across organization

**Documentation:**
- Central documentation portal
- Standardized templates
- Centralized governance

#### Federated Model

**Characteristics:**
- Documentation owned by teams
- Distributed review process
- Standards enforced via automation

**Documentation:**
- Team-specific documentation
- Shared templates and standards
- Automated quality checks

**Template:** See `templates/tier-enterprise/GOVERNANCE_MODEL.md`

---

## 9. Related Documents

| Document | Purpose |
|----------|---------|
| [Database Documentation](./19-DATABASE_DOCUMENTATION.md) | Data classification |
| [Governance](./07-GOVERNANCE.md) | Document ownership |
| [Operations](./06-OPERATIONS.md) | Incident documentation |
| [38-CONTEXT_GUIDANCE](./38-CONTEXT_GUIDANCE.md) | Enterprise-specific guidance |
| [example-enterprise-compliance.md](./examples/example-enterprise-compliance.md) | Compliance documentation example |

---

**Previous:** [23 - Data Pipelines](./23-DATA_PIPELINES.md)
**Next:** [25 - Infrastructure Code](./25-INFRASTRUCTURE_CODE.md)
