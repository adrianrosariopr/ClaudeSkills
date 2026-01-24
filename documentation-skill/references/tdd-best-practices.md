<overview>
Technical Design Documents (TDDs) describe how a system will be built. They translate product requirements into technical architecture, data models, and implementation plans. Without proper TDDs, teams face 47% higher risks of project failure due to poorly managed requirements.
</overview>

<core_sections>
## Essential TDD Sections

<section name="background">
**Background & Context**
- Why is this design needed?
- Link to PRD or requirements
- Current state of the system
- Business context driving the technical work
</section>

<section name="problem">
**Problem Statement**
- Technical challenge being solved
- Constraints and limitations
- What happens if we don't solve this?
</section>

<section name="goals">
**Goals and Non-Goals**
- **Goals**: What this design achieves
- **Non-Goals**: What this design explicitly does NOT address
- Prevents scope creep and sets expectations
</section>

<section name="solution">
**Proposed Solution**
- High-level approach (1-2 paragraphs)
- Key architectural decisions
- Why this approach over others
</section>

<section name="architecture">
**System Architecture**
- Component diagram showing relationships
- Data flow between components
- External dependencies
- **Must include at least one diagram**
</section>

<section name="detailed-design">
**Detailed Design**
- Component breakdowns
- API contracts and interfaces
- Data models and schemas
- Error handling strategies
- State management
</section>

<section name="alternatives">
**Alternatives Considered**
- At least 2-3 other approaches
- Pros and cons of each
- Why they were rejected
- **Critical for future maintainers**
</section>

<section name="data">
**Data Model**
- Database schema
- Data migrations required
- Data validation rules
- Storage and retention policies
</section>

<section name="api">
**API Design**
- Endpoint specifications
- Request/response contracts
- Authentication and authorization
- Rate limiting and quotas
- Error codes and messages
</section>

<section name="security">
**Security Considerations**
- Authentication/authorization requirements
- Data protection measures
- Potential vulnerabilities and mitigations
- Compliance requirements (GDPR, HIPAA, etc.)
</section>

<section name="testing">
**Testing Strategy**
- Unit testing approach
- Integration testing plan
- End-to-end testing
- Performance testing requirements
- Load testing targets
</section>

<section name="rollout">
**Rollout Plan**
- Deployment strategy (blue-green, canary, etc.)
- Feature flags
- Rollback procedure
- Monitoring and alerting
- Success criteria for rollout
</section>

<section name="open-questions">
**Open Questions**
- Unresolved decisions needing input
- Dependencies on other teams
- Risks requiring discussion
</section>
</core_sections>

<diagrams>
## Essential Diagrams

TDDs without diagrams are incomplete. Include at minimum:

<diagram name="architecture">
**Architecture Diagram**
Shows components and their relationships
```
┌─────────────┐     ┌─────────────┐
│   Client    │────▶│  API Gateway│
└─────────────┘     └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
       ┌──────────┐ ┌──────────┐ ┌──────────┐
       │ Service A│ │ Service B│ │ Service C│
       └────┬─────┘ └────┬─────┘ └────┬─────┘
            │            │            │
            └────────────┼────────────┘
                         ▼
                   ┌──────────┐
                   │ Database │
                   └──────────┘
```
</diagram>

<diagram name="sequence">
**Sequence Diagram**
Shows request/response flow between components
```
Client          API           Service        Database
  │              │               │              │
  │──Request────▶│               │              │
  │              │──Validate────▶│              │
  │              │               │──Query──────▶│
  │              │               │◀─Result──────│
  │              │◀─Response─────│              │
  │◀─Response────│               │              │
```
</diagram>

<diagram name="data-flow">
**Data Flow Diagram**
Shows how data moves through the system
</diagram>

<diagram name="state">
**State Diagram**
For stateful components (order status, user lifecycle)
</diagram>

Tools: ASCII art for simple diagrams, Mermaid for version control, Lucidchart/Excalidraw for complex diagrams.
</diagrams>

<good_vs_bad>
## Good TDD vs Bad TDD

<comparison name="alternatives">
**Bad Alternatives Section:**
"We considered other approaches but chose this one."

**Good Alternatives Section:**
"**Option A: Microservices**
- Pros: Independent scaling, team autonomy
- Cons: Operational complexity, network latency
- Rejected because: Team size (3 engineers) doesn't justify overhead

**Option B: Monolith with modules** (CHOSEN)
- Pros: Simpler deployment, easier debugging
- Cons: Scaling requires full app scaling
- Chosen because: Matches team size, can extract services later

**Option C: Serverless**
- Pros: Zero ops, auto-scaling
- Cons: Cold starts, vendor lock-in
- Rejected because: Latency requirements (< 100ms) conflict with cold starts"
</comparison>

<comparison name="api">
**Bad API Design:**
"API will have endpoints for user operations"

**Good API Design:**
```
POST /api/v1/users
Request:
{
  "email": "string (required, valid email)",
  "name": "string (required, 1-100 chars)"
}
Response 201:
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "created_at": "ISO8601"
}
Response 400:
{
  "error": "validation_error",
  "details": [{"field": "email", "message": "Invalid format"}]
}
```
</comparison>

<comparison name="rollout">
**Bad Rollout Plan:**
"We will deploy to production"

**Good Rollout Plan:**
"**Phase 1:** Deploy to staging, run full test suite
**Phase 2:** Canary release to 5% of users for 24 hours
**Phase 3:** Monitor error rates (<0.1%) and latency (<100ms p99)
**Phase 4:** Gradual rollout: 25% → 50% → 100% over 3 days
**Rollback trigger:** Error rate >0.5% OR latency >200ms p99
**Rollback procedure:** Revert deployment, no data migration needed"
</comparison>
</good_vs_bad>

<deliberation>
## The Deliberation Section

Often overlooked but critical. Documents:

- **Trade-offs accepted**: What you're giving up
- **Assumptions made**: What must be true for this to work
- **Risks identified**: What could go wrong
- **Decision rationale**: Why this path, not others

This section is gold for future maintainers trying to understand "why was it built this way?"
</deliberation>

<key_principles>
## Key Principles

1. **Diagrams are mandatory**: No TDD is complete without architecture diagrams
2. **Document alternatives**: Show you considered other approaches
3. **Explain trade-offs**: Future you will thank present you
4. **Security is not optional**: Address it explicitly
5. **Plan the rollout**: Deployment is part of the design
6. **Keep it updated**: Stale TDDs mislead engineers
7. **Link to PRD**: Connect technical decisions to business requirements
</key_principles>
