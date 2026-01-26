# Spec Writing Best Practices

<overview>
Guidelines for writing effective specifications that serve non-technical stakeholders (Product Management, Product Designers) and LLMs.
</overview>

<audience_first priority="critical">

## Write for Non-Technical Readers

**The primary audience is Product Management and Product Designers - NOT engineers.**

Every sentence should be understandable by someone with zero technical background. This is the most important principle in spec writing.

### Language Rules

| Instead of... | Write... |
|---------------|----------|
| "API endpoints" | "How the app gets and sends data" |
| "Database schema" | "What information is stored" |
| "Authentication flow" | "How users log in" |
| "State management" | "How the app remembers things" |
| "Middleware" | "Behind-the-scenes processing" |
| "Caching" | "Saved copies for faster loading" |
| "Async operations" | "Background tasks" |
| "Webhook" | "Automatic notification" |
| "CRUD operations" | "Create, view, edit, and delete [thing]" |
| "Pagination" | "Showing results in pages" |

### What to Include

- **User experiences**: What users see, click, and feel
- **Business outcomes**: What value features deliver
- **Visual descriptions**: What screens look like
- **Step-by-step flows**: What happens in order

### What to Exclude

- Code snippets
- Technical architecture diagrams
- Implementation details
- Framework-specific terminology
- Configuration specifics
- Developer-focused patterns

### Test Your Writing

Ask: "Would my non-technical manager understand this?"

If no, simplify. If you MUST use a technical term, explain it in one plain sentence.

</audience_first>

<core_principles>

## 1. Verifiable Requirements

Every functional requirement must be testable:

**Bad:**
```
FR-001: The system should be user-friendly.
```

**Good:**
```
FR-001: Users can complete checkout in 3 steps or fewer.
Acceptance: Given a cart with items, when user clicks checkout,
then they see: 1) Address, 2) Payment, 3) Confirm - and order is placed.
```

**Verification methods:**
- **Test** - Automated test can verify
- **Inspection** - Code review can verify
- **Demo** - Manual demonstration can verify
- **Analysis** - Calculation/analysis can verify

---

## 2. Separate What from How

The spec describes WHAT the system does, not HOW it's implemented:

**Bad (implementation detail):**
```
FR-010: Use Redux Toolkit with createSlice for state management.
```

**Good (behavior):**
```
FR-010: Cart state persists across page refreshes.
Acceptance: Given items in cart, when page is refreshed,
then cart items remain with correct quantities.
```

---

## 3. Prioritize Requirements

Use MoSCoW or similar:
- **Must** - Critical for launch
- **Should** - Important but not blocking
- **Could** - Nice to have
- **Won't** - Explicitly out of scope (this release)

---

## 4. Complete NFR Coverage

Non-functional requirements often forgotten:

**Performance:**
- Page load time targets
- API response time targets
- Concurrent user capacity

**Security:**
- Authentication requirements
- Data encryption requirements
- Input validation requirements

**Accessibility:**
- WCAG level target
- Keyboard navigation
- Screen reader support

**Reliability:**
- Uptime target
- Backup/recovery requirements
- Error handling requirements

</core_principles>

<writing_guidelines>

## Clarity

- Use active voice: "System sends email" not "Email is sent by system"
- Be specific: "within 200ms" not "quickly"
- Avoid jargon unless defined
- One requirement per statement

## Consistency

- Use same terminology throughout (define in glossary if needed)
- Same format for all FRs and NFRs
- Same acceptance criteria style (Given/When/Then recommended)

## Completeness

- Cover happy paths AND error cases
- Include edge cases that matter
- Document what's NOT included (non-goals)

## Traceability

- Number all requirements (FR-001, NFR-001)
- Link requirements to features/epics
- Reference related requirements

</writing_guidelines>

<obsidian_optimization>

## Obsidian-Friendly Formatting

**Frontmatter for queries:**
```yaml
---
title: Project Spec
type: project-spec
project: my-project
status: draft
tags: [project-spec, webapp, laravel]
---
```

**Wikilinks for connections:**
```markdown
Related: [[my-project-roadmap]], [[my-project-architecture]]
```

**Callouts for important notes:**
```markdown
> [!warning] Security Consideration
> All user input must be sanitized before database storage.

> [!info] Implementation Note
> This feature depends on the Stripe integration being complete.
```

**Dataview-queryable properties:**
```yaml
priority: must
status: implemented
owner: "@backend-team"
```

</obsidian_optimization>

<common_mistakes>

## Mistakes to Avoid

**Vague requirements:**
- "System should be fast" → Define specific targets
- "Intuitive UI" → Define specific behaviors
- "Handle errors gracefully" → Define error states and messages

**Missing context:**
- Requirements without user context
- Technical constraints not documented
- Assumptions not stated

**Scope creep in writing:**
- Adding features while documenting
- Mixing current state with future vision
- Including implementation details

**Inconsistent detail:**
- Some features over-specified
- Others under-specified
- Inconsistent acceptance criteria format

</common_mistakes>

<llm_readability>

## Optimizing for LLM Consumption

When Claude (or another LLM) reads this spec in the future, help it understand:

**Clear structure:**
- Consistent section organization
- Numbered requirements
- Tables for matrices (permissions, compatibility)

**Explicit relationships:**
- "FR-015 depends on FR-010"
- "This section covers authentication; see Data section for user storage"

**Context markers:**
- "As of [date], the current implementation..."
- "This differs from industry standard because..."

**Decision rationale:**
- Why certain approaches were chosen
- What alternatives were considered
- What constraints drove decisions

</llm_readability>

<tips_and_tricks>

## Pro Tips

1. **Start with user journeys** - Map the main flows first, then derive requirements

2. **Interview the code** - Read key files, comments, and tests to understand intent

3. **Check TODOs and FIXMEs** - They reveal known gaps and future plans

4. **Read commit history** - Recent commits show active development areas

5. **Document tribal knowledge** - Things obvious to current devs, invisible to newcomers

6. **Include the "why"** - Future readers need context, not just rules

7. **Version the spec** - Track changes as project evolves

8. **Link to evidence** - Reference specific files, tests, or configs that prove requirements

</tips_and_tricks>
