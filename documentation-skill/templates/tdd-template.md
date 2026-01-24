# {{SYSTEM_NAME}} - Technical Design Document

---
**Version:** {{VERSION}}
**Status:** Draft | In Review | Approved
**Author:** {{AUTHOR}}
**Last Updated:** {{DATE}}
**Reviewers:** {{REVIEWER_LIST}}

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | {{DATE}} | {{AUTHOR}} | Initial draft |

---

## 1. Background & Context

### Business Context
{{Why is this work needed? Link to PRD or business requirements.}}

**Related Documents:**
- PRD: {{Link}}
- Epic/Ticket: {{Link}}

### Current State
{{How does the system work today? What exists?}}

### Problem Statement
{{What technical challenge are we solving?}}

## 2. Goals and Non-Goals

### Goals
- {{Technical goal 1 - what this design achieves}}
- {{Technical goal 2}}
- {{Technical goal 3}}

### Non-Goals
- {{What this design explicitly does NOT address}}
- {{Deferred to future work}}

## 3. Proposed Solution

### Overview
{{High-level description of the approach (2-3 paragraphs)}}

### Key Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| {{Decision area}} | {{What we chose}} | {{Why}} |
| {{Decision area}} | {{What we chose}} | {{Why}} |

## 4. System Architecture

### Architecture Diagram

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────▶│     API     │────▶│  Database   │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
              ┌─────────┐   ┌─────────┐
              │ Service │   │  Cache  │
              │    A    │   │         │
              └─────────┘   └─────────┘
```

{{Replace with actual architecture diagram}}

### Components

| Component | Responsibility | Technology |
|-----------|---------------|------------|
| {{Component}} | {{What it does}} | {{Tech stack}} |
| {{Component}} | {{What it does}} | {{Tech stack}} |

### External Dependencies
- {{External service/API}}
- {{Third-party library}}

## 5. Detailed Design

### Component: {{Component Name}}

**Purpose:** {{What this component does}}

**Interface:**
```{{language}}
// {{Interface definition}}
```

**Behavior:**
- {{How it works - key behavior 1}}
- {{Key behavior 2}}
- {{Error handling}}

### Component: {{Component Name}}

{{Repeat for each major component}}

## 6. Data Model

### Schema

```sql
-- {{Table/Collection name}}
CREATE TABLE {{table_name}} (
    id          UUID PRIMARY KEY,
    {{field}}   {{TYPE}} NOT NULL,
    {{field}}   {{TYPE}},
    created_at  TIMESTAMP DEFAULT NOW(),
    updated_at  TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_{{name}} ON {{table}}({{field}});
```

### Data Flow

```
[Input] → [Validation] → [Processing] → [Storage] → [Response]
```

### Migrations
| Migration | Description | Reversible |
|-----------|-------------|------------|
| {{Migration name}} | {{What it does}} | Yes/No |

## 7. API Design

### Endpoints

#### {{METHOD}} {{/path}}

**Description:** {{What this endpoint does}}

**Request:**
```json
{
  "{{field}}": "{{type}} (required) - {{description}}",
  "{{field}}": "{{type}} (optional) - {{description}}"
}
```

**Response 200:**
```json
{
  "{{field}}": "{{type}}",
  "{{field}}": "{{type}}"
}
```

**Response 400:**
```json
{
  "error": "{{error_code}}",
  "message": "{{Human-readable message}}",
  "details": []
}
```

**Response 401/403/404/500:** {{Brief description}}

#### {{METHOD}} {{/path}}

{{Repeat for each endpoint}}

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| {{ERROR_CODE}} | {{4XX/5XX}} | {{When this occurs}} |

## 8. Security Considerations

### Authentication & Authorization
- **Authentication:** {{Method - OAuth, API Key, JWT, etc.}}
- **Authorization:** {{How permissions are checked}}

### Data Protection
- **At Rest:** {{Encryption method}}
- **In Transit:** {{TLS version, etc.}}
- **PII Handling:** {{How sensitive data is handled}}

### Threat Model

| Threat | Mitigation |
|--------|------------|
| {{Threat description}} | {{How addressed}} |
| {{Threat description}} | {{How addressed}} |

## 9. Alternatives Considered

### Option A: {{Name}}

**Description:** {{Brief description}}

**Pros:**
- {{Advantage 1}}
- {{Advantage 2}}

**Cons:**
- {{Disadvantage 1}}
- {{Disadvantage 2}}

**Why Rejected:** {{Reason}}

### Option B: {{Name}} (CHOSEN)

**Description:** {{Brief description}}

**Pros:**
- {{Advantage 1}}
- {{Advantage 2}}

**Cons:**
- {{Disadvantage 1 (accepted trade-off)}}

**Why Chosen:** {{Reason}}

### Option C: {{Name}}

{{Repeat pattern}}

## 10. Testing Strategy

### Unit Tests
- {{What will be unit tested}}
- Coverage target: {{X}}%

### Integration Tests
- {{Integration test scenarios}}
- {{External service mocking approach}}

### End-to-End Tests
- {{E2E test scenarios}}

### Performance Tests
- Load test: {{Target RPS}}
- Latency test: {{Target p99}}

## 11. Rollout Plan

### Deployment Strategy
{{Blue-green / Canary / Rolling / Feature flags}}

### Phases

| Phase | % Traffic | Duration | Success Criteria |
|-------|-----------|----------|------------------|
| 1. Canary | 5% | 24 hours | Error rate < 0.1% |
| 2. Partial | 25% | 48 hours | Latency p99 < 100ms |
| 3. Full | 100% | - | All metrics green |

### Rollback Plan
**Trigger:** {{When to rollback - metrics thresholds}}

**Procedure:**
1. {{Rollback step 1}}
2. {{Rollback step 2}}
3. {{Verification step}}

### Monitoring & Alerting

| Metric | Alert Threshold | Action |
|--------|-----------------|--------|
| Error rate | > {{X}}% | {{Response}} |
| Latency p99 | > {{X}}ms | {{Response}} |
| {{Custom metric}} | {{Threshold}} | {{Response}} |

## 12. Open Questions

- [ ] {{Question 1}} - Owner: {{Name}} - Due: {{Date}}
- [ ] {{Question 2}} - Owner: {{Name}} - Due: {{Date}}

## 13. Future Work

Items explicitly deferred from this design:
- {{Future enhancement 1}}
- {{Future enhancement 2}}

## 14. Appendix

### References
- {{Link to relevant documentation}}
- {{Link to external resources}}

### Glossary

| Term | Definition |
|------|------------|
| {{Term}} | {{Definition}} |

---

## Approval

| Role | Name | Date | Approved |
|------|------|------|----------|
| Tech Lead | | | ☐ |
| Security | | | ☐ |
| Architecture | | | ☐ |
