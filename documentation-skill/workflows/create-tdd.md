# Workflow: Create Technical Design Document

<required_reading>
**Read these reference files NOW before creating the TDD:**
1. references/tdd-best-practices.md
2. references/llm-optimized-structure.md
3. references/anti-patterns.md
</required_reading>

<process>

<step name="gather-context">
## Step 1: Gather Context

Ask the user for essential information:

**Required:**
- What system/feature is being designed?
- What problem does it solve technically?
- Is there a PRD or requirements doc to reference?

**If not provided, ask:**
- What are the technical constraints?
- What's the existing system architecture?
- What are the performance requirements?
- Who will implement this?
- What's the timeline?
</step>

<step name="read-template">
## Step 2: Read Template

Read `templates/tdd-template.md` and understand the structure.

The template provides standard sections. You'll fill each based on the technical requirements.
</step>

<step name="draft-tdd">
## Step 3: Draft the TDD

Fill each section of the template:

**1. Background & Context** - Why this design is needed, link to PRD
**2. Problem Statement** - Technical challenge being solved
**3. Goals & Non-Goals** - What this design will and won't achieve
**4. Proposed Solution** - High-level approach
**5. System Architecture** - Diagrams showing component relationships
**6. Detailed Design** - Component breakdowns, APIs, data models
**7. Alternatives Considered** - Other approaches and why rejected
**8. Data Model** - Schema, migrations, data flow
**9. API Design** - Endpoints, contracts, error handling
**10. Security Considerations** - Auth, data protection, vulnerabilities
**11. Testing Strategy** - Unit, integration, E2E approach
**12. Rollout Plan** - Deployment strategy, feature flags, rollback
**13. Open Questions** - Unresolved decisions needing input

**Writing principles:**
- Include architecture diagrams (ASCII or links to diagrams)
- Document trade-offs and decision rationale
- Specify error handling and edge cases
- Define contracts between components
- Keep it technical but accessible to the team
</step>

<step name="add-diagrams">
## Step 4: Add Diagrams

TDDs need visual communication. Include:

- **Architecture diagram**: Components and their relationships
- **Sequence diagram**: Request/response flows
- **Data flow diagram**: How data moves through the system
- **State diagram**: For stateful components

Use ASCII diagrams for simplicity or link to external tools (Mermaid, Lucidchart, Excalidraw).

```
Example ASCII architecture:
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Client │────▶│   API   │────▶│   DB    │
└─────────┘     └─────────┘     └─────────┘
                     │
                     ▼
               ┌─────────┐
               │  Cache  │
               └─────────┘
```
</step>

<step name="document-alternatives">
## Step 5: Document Alternatives

For each major decision, document:

1. **Options considered** - At least 2-3 alternatives
2. **Pros/cons of each** - Honest assessment
3. **Why this choice** - Decision rationale
4. **Trade-offs accepted** - What you're giving up

This section is critical for future maintainers understanding "why".
</step>

<step name="review-and-refine">
## Step 6: Review and Refine

Check the TDD against these criteria:

- [ ] Could another engineer implement this without asking questions?
- [ ] Are all components and their interactions documented?
- [ ] Is there an architecture diagram?
- [ ] Are alternatives documented with trade-offs?
- [ ] Is the testing strategy defined?
- [ ] Is the rollout plan realistic?
- [ ] Are security implications addressed?

Ask user: "Would you like me to expand the technical details in any section?"
</step>

<step name="finalize">
## Step 7: Finalize

- Add version number and date
- Mark "Open Questions" for team discussion
- Include author and reviewers
- Add revision history placeholder
</step>

</process>

<anti_patterns>
Avoid these common TDD mistakes:
- **No diagrams**: Text-only TDDs are harder to understand
- **Missing alternatives**: Looks like the first idea was used without thought
- **No rationale**: Future maintainers won't understand "why"
- **Overloading detail**: Implementation code belongs in code, not docs
- **Stale content**: Outdated TDDs mislead engineers
- **Skipping security**: Security is not an afterthought
- **No rollout plan**: Deploy strategies prevent incidents
</anti_patterns>

<success_criteria>
A complete TDD:
- [ ] Clearly defines the technical problem
- [ ] Includes architecture diagram(s)
- [ ] Documents alternatives considered with rationale
- [ ] Specifies data models and API contracts
- [ ] Addresses security considerations
- [ ] Has a testing strategy
- [ ] Includes rollout/deployment plan
- [ ] Lists open questions for discussion
- [ ] Is implementable without ambiguity
</success_criteria>
