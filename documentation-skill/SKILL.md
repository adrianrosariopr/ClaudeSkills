---
name: documentation-skill
description: Create and optimize Product Requirement Documents (PRDs), Technical Design Documents (TDDs), and Game Design Documents (GDDs). Full lifecycle - draft, review, optimize for human and LLM readability. Use when writing or improving any product, technical, or game design documentation.
---

<essential_principles>

<principle name="documents-are-living">
Documents are living artifacts, not one-time deliverables. Structure them for easy updates. Include version history. Mark sections as draft/approved/deprecated. A stale document is worse than no document.
</principle>

<principle name="audience-first">
Know your readers before writing. Engineers need different detail than executives. 65% of readers only need specific sections - make navigation effortless with clear hierarchy, table of contents, and semantic headings.
</principle>

<principle name="dual-readability">
Optimize for both humans AND machines. Use hierarchical headers (H1 → H2 → H3), short paragraphs (3-5 sentences), bulleted lists, and tables. This structure aids LLM parsing while improving human scanability.
</principle>

<principle name="right-level-of-detail">
Balance specificity and flexibility. Too vague = misinterpretation. Too detailed = unread documents. Quantify requirements ("loads in < 500ms") instead of vague adjectives ("fast"). Include decision rationale, not just decisions.
</principle>

<principle name="visual-communication">
Diagrams, flowcharts, and mockups convey complex ideas better than text. Include architecture diagrams in TDDs, user flows in PRDs, and gameplay loops in GDDs. A picture is worth a thousand words of requirements.
</principle>

</essential_principles>

<intake>
What would you like to do?

1. **Create a PRD** - Product Requirements Document for a feature or product
2. **Create a TDD** - Technical Design Document for implementation
3. **Create a GDD** - Game Design Document for a game or game feature
4. **Audit a document** - Review existing document for quality and completeness
5. **Optimize for LLM** - Make a document more machine-readable
6. **Something else** - General documentation guidance

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "PRD", "product", "requirements", "feature spec" | `workflows/create-prd.md` |
| 2, "TDD", "technical", "design doc", "architecture" | `workflows/create-tdd.md` |
| 3, "GDD", "game", "game design" | `workflows/create-gdd.md` |
| 4, "audit", "review", "check", "improve" | `workflows/audit-document.md` |
| 5, "LLM", "optimize", "machine", "AI readable" | `workflows/optimize-for-llm.md` |
| 6, other | Clarify intent, then select appropriate workflow |

**After reading the workflow, follow it exactly.**
</routing>

<reference_index>
All domain knowledge in `references/`:

**Document Types:**
- prd-best-practices.md - PRD structure, sections, what makes good/bad PRDs
- tdd-best-practices.md - TDD structure, sections, what makes good/bad TDDs
- gdd-best-practices.md - GDD structure, sections, what makes good/bad GDDs

**Cross-Cutting:**
- llm-optimized-structure.md - Structure patterns for LLM/AI parsing
- anti-patterns.md - Common mistakes across all document types
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| create-prd.md | Create Product Requirements Document from scratch |
| create-tdd.md | Create Technical Design Document from scratch |
| create-gdd.md | Create Game Design Document from scratch |
| audit-document.md | Review and improve existing documentation |
| optimize-for-llm.md | Optimize document structure for LLM consumption |
</workflows_index>

<templates_index>
| Template | Purpose |
|----------|---------|
| prd-template.md | Standard PRD structure with placeholders |
| tdd-template.md | Standard TDD structure with placeholders |
| gdd-template.md | Standard GDD structure with placeholders |
</templates_index>

<success_criteria>
A well-crafted document:
- Has clear hierarchy with navigable sections
- Answers "what" and "why", not just "how"
- Uses quantified requirements, not vague adjectives
- Includes visual aids where appropriate
- Is findable, readable, and up-to-date
- Works for both human readers and LLM parsing
</success_criteria>
