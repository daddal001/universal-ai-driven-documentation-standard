---
title: "AI Agents Documentation Guide"
type: "standard"
status: "approved"
classification: "public"
owner: "@documentation-maintainer"
created: "2025-12-09"
last_updated: "2025-12-12"
version: "1.1.0"
---



> This section ensures AI-generated documentation matches human standards. **All AI assistants (Claude, codex, Cursor, etc.) must follow these guidelines.**

---

## Table of Contents

1. [AI Quality Checklist](#ai-quality-checklist)
2. [llms.txt Standard](#llmstxt-standard)
3. [MCP Server Integration](#mcp-server-integration)
4. [RAG Optimization](#rag-optimization)
5. [Common AI Mistakes](#common-ai-mistakes)
6. [Document Type Selection](#document-type-selection)
7. [AI Agent Contribution Rules](#ai-agent-contribution-rules)

---

## AI Quality Checklist

Before submitting AI-generated documentation:

- [ ] Verified all file paths exist in codebase
- [ ] Verified all function/class names match actual code
- [ ] Verified all version numbers match package manifests
- [ ] Used templates from `docs/templates/` as base
- [ ] Cross-referenced instead of duplicating content
- [ ] Included realistic examples (not `foo`/`bar`)
- [ ] Documented error handling and edge cases
- [ ] Tested code examples for correctness
- [ ] Added AI-generated comment with date
- [ ] Requested human review for architectural content

### Accuracy Standard

AI assistants are held to the **same 100% accuracy standard** as human developers. Documentation with hallucinated content will be rejected:

| ❌ Hallucinated | ✅ Verified |
|-----------------|-------------|
| Invented version numbers | Check `package.json` or `pyproject.toml` |
| Non-existent features | Verify against actual code |
| Wrong file paths | Confirm paths exist |
| Made-up configurations | Test actual configs |

---

## llms.txt Standard

Create an `llms.txt` file at the repository root to help AI agents understand the project:

```txt
# Project Name

> One-line description

## Overview
Brief project description for LLM context.

## Documentation Entry Points
- /docs/INDEX.md - Documentation index
- /docs/ARCHITECTURE.md - System architecture
- /docs/api/INDEX.md - API documentation
- /README.md - Project overview

## Quick Facts
- Languages: Python, TypeScript
- Frameworks: FastAPI, Next.js
- API Style: REST + SSE
- Test Framework: pytest, vitest

## Key Patterns
- Authentication: JWT via HTTPOnly cookies
- Error format: { "error": string, "code": number }
- Rate limiting: 100 requests/minute
```

**Purpose:** Machine-readable project summary for LLMs. Text-based, easy to parse, provides context.

---

## MCP Server Integration

The Model Context Protocol (MCP) allows AI tools to interact with documentation systems directly. Expose your docs as MCP resources:

### mcp-docs-server.json

```json
{
  "name": "project-documentation",
  "version": "1.0.0",
  "resources": [
    {
      "name": "architecture",
      "uri": "docs://architecture/*",
      "description": "System architecture and design decisions"
    },
    {
      "name": "api-reference",
      "uri": "docs://api/*",
      "description": "API documentation and OpenAPI specs"
    },
    {
      "name": "runbooks",
      "uri": "docs://operations/runbooks/*",
      "description": "Operational runbooks for incident response"
    }
  ],
  "tools": [
    {
      "name": "search_docs",
      "description": "Semantic search across all documentation"
    },
    {
      "name": "get_runbook",
      "description": "Retrieve specific runbook by alert name"
    }
  ]
}
```

### Benefits of MCP Protocol

| Benefit | Description |
|---------|-------------|
| **Real-time context** | AI agents get current docs, not cached |
| **Structured access** | Agents access specific doc types |
| **Tool integration** | Docs become callable resources |

---

## IDE codex Instructions

Optimize AI code assistants (GitHub codex, Cursor, etc.) with project-specific instructions.

### .github/codex-instructions.md

Create this file to guide code generation:

```markdown
# codex Instructions

## Project Context
This is an enterprise platform using FastAPI (Python) and Next.js (TypeScript).

## Code Patterns
- Use async/await for all I/O operations
- Type hints required on all functions
- Use Pydantic for request/response models
- Error responses follow: `{ "error": string, "code": number, "docRef": string }`

## Naming Conventions
- Python: snake_case for functions/variables
- TypeScript: camelCase for functions, PascalCase for components
- Database: snake_case for tables/columns

## Architecture Rules
- Services communicate via REST, not direct imports
- All API endpoints require authentication
- Rate limiting: 100 req/min per user

## Testing
- Pytest for Python, Vitest for TypeScript
- Minimum 80% coverage required
- Use factories for test data, not fixtures
```

### Feature-Specific Instructions

For complex features, add `.md` files in relevant directories:

```text
services/auth/
├── src/
├── docs/
│   └── codex-context.md    # Auth-specific patterns
└── README.md
```

---

## Semantic Linking & Knowledge Graph Readiness

Structure documentation for knowledge graph indexing and semantic search.

### Required Frontmatter Tags

Add semantic metadata to frontmatter:

```yaml
---
title: "Rate Limiting Configuration"
type: "guide"
# Semantic tagging
concepts:
  - "rate-limiting"
  - "api-throttling"
  - "request-quotas"
related_ids:
  - "OPS-429"      # Error code
  - "ARCH-015"     # Architecture decision
  - "SVC-gateway"  # Service catalog ID
related_docs:
  - "./06-OPERATIONS.md#rate-limiting"
  - "../runbooks/RATE_LIMIT_EXCEEDED.md"
---
```

### Semantic Linking Requirements

| Requirement | Purpose |
|-------------|---------|
| **concepts** | Terms for semantic search matching |
| **related_ids** | Cross-reference to errors, ADRs, services |
| **related_docs** | Explicit links to related documentation |

### Benefits for AI Agents

| Feature | How It Helps |
|---------|--------------|
| **Concept tags** | AI finds docs by meaning, not just keywords |
| **Cross-references** | AI can follow relationship chains |
| **Structured metadata** | Enables knowledge graph construction |

---

## RAG Optimization

To ensure documentation is effectively retrieved by AI agents using RAG (Retrieval-Augmented Generation):

### Structure Guidelines

| Guideline | Description |
|-----------|-------------|
| **Chunking-friendly** | Use frequent, descriptive headers (H2, H3) |
| **Short paragraphs** | Avoid unbroken blocks >500 words |
| **Contextual headers** | `## Gateway Configuration` not `## Config` |
| **Self-contained sections** | Each section makes sense in isolation |
| **Keywords** | Include synonyms and domain terms naturally |

### Every Page Is Page One

Each section should:

- Be understandable without reading previous sections
- Avoid "See above" references
- Repeat context briefly when needed
- Include necessary imports/setup in code examples

### Q&A Format

For FAQs, use question-and-answer format—exceptionally powerful for RAG:

````markdown
### Q: How do I configure rate limiting?

**A:** Add to `config.py`:
```python
RATE_LIMIT = "100/minute"
```
````

---

## Common AI Mistakes

| Mistake | Why It's Wrong | How to Avoid |
|---------|----------------|--------------|
| Inventing version numbers | Causes confusion, breaks builds | Always check package files |
| Documenting non-existent features | Hallucination confuses developers | Verify against actual code |
| Copy-pasting between files | Violates SSOT, causes drift | Use cross-references |
| Generic examples (`foo`, `bar`) | Not realistic, hard to understand | Use domain-relevant examples |
| Missing error documentation | Incomplete, frustrates users | Document all error paths |
| Outdated file paths | Breaks navigation | Verify paths exist |

---

## Document Type Selection

Use this decision tree to determine which document type to generate:

```text
START
│
├─ Is this a folder/directory?
│   └─ YES → Generate: README.md
│
├─ Is this describing how to respond to an incident?
│   ├─ Is it a security incident? → Generate: Security Playbook
│   ├─ Is it an operational incident? → Generate: Runbook
│   └─ Is it post-incident analysis? → Generate: Incident Report
│
├─ Is this describing a technical decision?
│   ├─ Implementation time > 1 week? → Generate: ADR
│   └─ Implementation time < 1 week? → Add to: Decision Log
│
├─ Is this describing an API?
│   └─ YES → Generate: API Documentation (+ OpenAPI spec)
│
├─ Is this describing system design?
│   └─ YES → Generate: Architecture Document
│
└─ DEFAULT → Generate: README.md
```

---

## AI Agent Contribution Rules

### Required Behaviors

1. **Follow all structural requirements** — No exceptions to document type definitions
2. **Verify facts against actual code** — Never hallucinate features, versions, or paths
3. **Use cross-references** — Never duplicate content across files
4. **Mark significant contributions**:

    ```markdown
   <!-- AI-generated: [agent-name] [YYYY-MM-DD] - Reviewed by: [pending|@username] -->
    ```

5. **Preserve ownership** — Never modify the `owner` field in frontmatter
6. **Request human review** — All architectural documentation requires human approval

### AI Content Provenance

For substantial AI-generated documentation, include provenance metadata in frontmatter:

```yaml
---
title: "Service Documentation"
ai_generated: true
ai_metadata:
  model: "claude-3.5-sonnet"
  model_version: "2024-10-22"
  generation_date: "2025-12-09"
  prompt_context: "Generated from codebase analysis"
  confidence_score: 0.95
  human_reviewer: "@username"
  review_date: "2025-12-10"
  review_status: "approved"  # pending|approved|rejected
# Semantic Graphing
concepts:
  - "rate-limiting"
  - "authentication"
related_ids:
  - "OPS-001"
  - "ARCH-102"
---
```

**Confidence Scoring Guide:**

| Score | Meaning | Review Required |
|-------|---------|-----------------|
| 0.9+ | High confidence, facts verified | Spot check only |
| 0.7-0.9 | Medium confidence | Full review |
| <0.7 | Low confidence | Rewrite recommended |

### Prohibited Behaviors

| ❌ Don't | Reason |
|----------|--------|
| Invent version numbers | Causes build failures |
| Document non-existent features | Confuses developers |
| Copy content between files | Violates SSOT |
| Use generic examples | Not realistic |
| Skip error documentation | Incomplete docs |
| Create orphaned documents | Must link from INDEX |

---

## 8. Agentic AI Protocols (2025)

### The Agentic AI Foundation Standards

The Linux Foundation's **Agentic AI Foundation (AAIF)**, established December 2025 by Anthropic, Block, and OpenAI, defines open standards for agent interoperability.

#### Supported Protocols

| Protocol | Purpose | When to Use |
|----------|---------|-------------|
| **MCP (Model Context Protocol)** | Tool and context integration | AI accessing docs, tools, APIs |
| **A2A (Agent2Agent)** | Agent-to-agent communication | Multi-agent workflows |
| **ACP (Agent Communication Protocol)** | Task invocation and lifecycle | Enterprise agent orchestration |

### AGENTS.md Standard

Create an `AGENTS.md` file at repository root for AI coding agents:

```markdown
# Agent Instructions

## Project Context
This is an enterprise platform using FastAPI (Python) and Next.js (TypeScript).

## Code Standards
- Use async/await for all I/O operations
- Type hints required on all functions
- Use Pydantic for request/response models
- Follow existing patterns in codebase

## Restricted Actions
- Do not modify CODEOWNERS without approval
- Do not change authentication/security code without review
- Do not delete migration files

## Common Tasks
- New endpoint: Follow pattern in `src/api/routes/`
- New model: Add to `src/models/` with migrations
- Tests: Add to `tests/` matching source structure
```

### Benefits of Adoption

| Benefit | Description |
|---------|-------------|
| **Reduced hallucinations** | Explicit context prevents invented features |
| **Consistent patterns** | Agents follow project conventions |
| **Industry adoption** | Supported by OpenAI, GitHub, major OSS projects |

---

## 9. Prompt Engineering for Documentation

### Pre-Built Prompts Library

Use these tested prompts for AI agents to generate consistent, high-quality documentation.

#### README Generation

```markdown
You are a documentation expert. Generate a README.md for the following codebase.

**Context:**
- Repository: {repo_name}
- Primary Language: {language}
- Framework: {framework}

**Requirements:**
1. Follow the structure in docs/standards/templates/README.md
2. Include: Overview, Quick Start, Prerequisites, Installation, Usage, Configuration, API Reference (if applicable), Contributing, License
3. All code examples must be copy-paste ready
4. Include realistic examples, not foo/bar placeholders
5. Verify all file paths exist before referencing

**Frontmatter:**
Include YAML frontmatter with: title, type, status, owner, created, last_updated, version

**Output:** Complete README.md content
```

#### API Documentation Prompt

```markdown
Generate OpenAPI 3.1 documentation for this API endpoint.

**Endpoint Details:**
- Method: {method}
- Path: {path}
- Handler: {handler_file}:{function_name}

**Requirements:**
1. Include summary, description, operationId
2. Document all request parameters and body schema
3. Document all response codes (200, 400, 401, 403, 404, 429, 500)
4. Include realistic examples for request/response bodies
5. Add rate limiting info if applicable
6. Reference error codes from docs/api/ERROR_CODES.md

**Output:** OpenAPI YAML specification
```

#### Runbook Generation Prompt

```markdown
Generate a runbook for the following operational scenario.

**Scenario:** {alert_name}
**Service:** {service_name}
**Typical Trigger:** {trigger_condition}

**Requirements:**
1. Follow structure: Trigger Conditions → Severity → Prerequisites → Steps → Expected Outcomes → Rollback → Escalation
2. Each step must be numbered and unambiguous
3. Include exact commands (copy-paste ready)
4. Include observability context: logs, metrics, traces
5. Reference existing runbooks for related scenarios
6. Add time estimates for each major step

**Frontmatter:** Include severity_trigger, last_tested, tested_by, next_test_due

**Output:** Complete runbook in Markdown
```

#### Architecture Document Prompt

```markdown
Generate an architecture document for this system component.

**Component:** {component_name}
**Type:** {microservice|library|infrastructure}

**Requirements:**
1. Include C4 Model diagrams (Level 2: Container)
2. Use Mermaid.js for all diagrams
3. Document: Purpose, Key Components, Data Flow, Dependencies, Security Boundaries
4. Include sequence diagrams for critical flows
5. Reference related ADRs
6. Add non-functional requirements: SLOs, scaling limits, failure modes

**Output:** Architecture document in Markdown with embedded Mermaid diagrams
```

### Prompt Chaining for Complex Docs

For large documentation tasks, chain prompts:

```text
1. ANALYZE: "Analyze {codebase_path} and list all public APIs, their purposes, and categorize by domain"
2. PLAN: "Create a documentation outline for these {n} APIs, grouping related endpoints"
3. GENERATE: "Generate detailed documentation for API group: {group_name}"
4. VERIFY: "Cross-reference generated docs against actual code. List any inaccuracies."
5. REFINE: "Fix inaccuracies and add missing edge cases"
```

### Prompt Variables

Standard variables for documentation prompts:

| Variable | Source | Example |
|----------|--------|---------|
| `{repo_name}` | Repository root | `your-project-name` |
| `{language}` | Detected from files | `TypeScript` |
| `{framework}` | package.json/requirements.txt | `Next.js` |
| `{service_name}` | Directory name | `backend-gateway` |
| `{owner}` | CODEOWNERS | `@backend-team` |
| `{version}` | package.json | `2.1.0` |

---

## Related Documents

| Document | Purpose |
|----------|---------|
| [Document Types](./03-DOCUMENT_TYPES.md) | All document types |
| [Quality](./05-QUALITY.md) | Quality standards |

---

**Previous:** [03 - Document Types](./03-DOCUMENT_TYPES.md)
**Next:** [05 - Quality](./05-QUALITY.md)
