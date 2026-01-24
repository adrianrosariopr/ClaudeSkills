# Workflow: Audit Existing Document

<required_reading>
**Read the relevant reference file based on document type:**
- PRD → references/prd-best-practices.md
- TDD → references/tdd-best-practices.md
- GDD → references/gdd-best-practices.md

**Always read:**
- references/anti-patterns.md
- references/llm-optimized-structure.md
</required_reading>

<process>

<step name="identify-document">
## Step 1: Identify Document Type

Ask user to provide the document or path to it.

Determine the document type:
- **PRD**: Product/feature requirements, user stories, success metrics
- **TDD**: Technical architecture, system design, API specs
- **GDD**: Game mechanics, core loops, narrative design
- **Hybrid**: Some docs combine elements

Load the appropriate best practices reference.
</step>

<step name="structural-audit">
## Step 2: Structural Audit

Check document structure:

**Hierarchy & Navigation**
- [ ] Clear header hierarchy (H1 → H2 → H3)?
- [ ] Table of contents or navigation?
- [ ] Logical section ordering?
- [ ] Sections are self-contained (can be read independently)?

**LLM Readability**
- [ ] Short paragraphs (3-5 sentences)?
- [ ] Bulleted/numbered lists for related items?
- [ ] Tables for structured data?
- [ ] Separate topics in distinct sections?

**Metadata**
- [ ] Version number and date?
- [ ] Author/owner identified?
- [ ] Status markers (Draft/Approved/Deprecated)?
- [ ] Change history or revision log?
</step>

<step name="content-audit">
## Step 3: Content Audit

Check content quality based on document type:

**For PRDs:**
- [ ] Problem statement clear?
- [ ] Success metrics measurable and quantified?
- [ ] User personas or stories defined?
- [ ] Requirements prioritized (Must/Should/Nice)?
- [ ] Scope defined (in and out)?
- [ ] Acceptance criteria for requirements?

**For TDDs:**
- [ ] Architecture diagrams included?
- [ ] Alternatives considered and documented?
- [ ] Decision rationale explained?
- [ ] API contracts defined?
- [ ] Security considerations addressed?
- [ ] Testing and rollout strategy?

**For GDDs:**
- [ ] Core loop clearly defined?
- [ ] Mechanics are concrete, not abstract?
- [ ] Visual references included?
- [ ] Target audience defined?
- [ ] Scope realistic for team?
</step>

<step name="anti-pattern-check">
## Step 4: Anti-Pattern Check

Scan for common problems:

**Vagueness**
- [ ] Are there vague adjectives ("fast", "easy", "intuitive")?
- [ ] Are requirements quantified where possible?
- [ ] Is "why" explained, not just "what"?

**Staleness**
- [ ] When was it last updated?
- [ ] Are there references to deprecated tech/features?
- [ ] Do any sections contradict current reality?

**Imbalance**
- [ ] Is any section disproportionately detailed/sparse?
- [ ] Are all stakeholder needs addressed?
- [ ] Is there excessive detail that belongs elsewhere?

**Missing Elements**
- [ ] Are there TODOs or placeholders left unfilled?
- [ ] Are there sections that say "TBD"?
- [ ] Are there broken links or references?
</step>

<step name="generate-report">
## Step 5: Generate Audit Report

Create a structured audit report:

```markdown
# Document Audit Report

**Document:** [Name]
**Type:** [PRD/TDD/GDD]
**Audited:** [Date]

## Summary
[1-2 sentence overall assessment]

## Strengths
- [What the document does well]
- [...]

## Issues Found

### Critical (Must Fix)
- [ ] [Issue]: [Why it matters] → [Suggested fix]

### Important (Should Fix)
- [ ] [Issue]: [Why it matters] → [Suggested fix]

### Minor (Nice to Fix)
- [ ] [Issue]: [Why it matters] → [Suggested fix]

## Recommendations
1. [Highest priority action]
2. [Second priority action]
3. [...]

## Score
- Structure: [Good/Needs Work/Poor]
- Content: [Good/Needs Work/Poor]
- Readability: [Good/Needs Work/Poor]
- Completeness: [Good/Needs Work/Poor]
```
</step>

<step name="offer-fixes">
## Step 6: Offer to Fix Issues

After presenting the report, ask:

"Would you like me to:
1. Fix the critical issues now?
2. Rewrite specific sections?
3. Provide a fully revised version?
4. Just use this report as a guide?"
</step>

</process>

<audit_checklist>
Quick reference checklist for all document types:

**Universal Requirements**
- [ ] Clear purpose statement
- [ ] Defined audience
- [ ] Logical structure with navigation
- [ ] Version and date
- [ ] Owner/author
- [ ] No stale content

**Writing Quality**
- [ ] Active voice
- [ ] Short paragraphs
- [ ] Quantified requirements
- [ ] Decision rationale included
- [ ] No jargon without definition

**LLM Optimization**
- [ ] Hierarchical headers
- [ ] Lists and tables for structured data
- [ ] Topics separated into sections
- [ ] Consistent formatting
</audit_checklist>

<success_criteria>
A thorough audit:
- [ ] Identifies document type correctly
- [ ] Checks structure, content, and anti-patterns
- [ ] Provides specific, actionable feedback
- [ ] Prioritizes issues (critical/important/minor)
- [ ] Offers concrete suggestions for improvement
- [ ] Gives overall assessment with scores
</success_criteria>
