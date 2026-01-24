# Workflow: Create Product Requirements Document

<required_reading>
**Read these reference files NOW before creating the PRD:**
1. references/prd-best-practices.md
2. references/llm-optimized-structure.md
3. references/anti-patterns.md
</required_reading>

<process>

<step name="gather-context">
## Step 1: Gather Context

Ask the user for essential information:

**Required:**
- What product/feature is this PRD for?
- What problem does it solve?
- Who is the target user?

**If not provided, ask:**
- What are the success metrics?
- What's the timeline/priority?
- Any constraints (budget, tech, regulatory)?
- Who are the stakeholders?
</step>

<step name="read-template">
## Step 2: Read Template

Read `templates/prd-template.md` and understand the structure.

The template provides the standard sections. You'll fill each placeholder based on the gathered context.
</step>

<step name="draft-prd">
## Step 3: Draft the PRD

Fill each section of the template:

**1. Overview** - 2-3 sentences capturing the essence
**2. Problem Statement** - The pain point with evidence/data
**3. Goals & Success Metrics** - Quantified, measurable outcomes
**4. User Personas** - Who benefits and how
**5. Requirements** - Functional and non-functional, prioritized
**6. User Stories** - "As a [user], I want [goal] so that [benefit]"
**7. Scope** - What's in, what's explicitly out
**8. Design & UX** - Mockups, flows, or links to designs
**9. Timeline & Milestones** - Phases with deliverables
**10. Risks & Mitigations** - What could go wrong

**Writing principles:**
- Use quantified requirements ("< 2 second load time") not vague ("fast")
- Include acceptance criteria for each requirement
- Mark priorities: Must-have / Should-have / Nice-to-have
- Keep paragraphs short (3-5 sentences)
- Use hierarchical headers for navigation
</step>

<step name="review-and-refine">
## Step 4: Review and Refine

Check the PRD against these criteria:

- [ ] Does it answer "what" and "why" clearly?
- [ ] Are success metrics specific and measurable?
- [ ] Is the scope clearly defined (in/out)?
- [ ] Are requirements prioritized?
- [ ] Is it navigable (clear sections, headers)?
- [ ] Would an engineer know what to build?
- [ ] Would a designer know what to design?

Ask user: "Would you like me to expand any section or adjust the detail level?"
</step>

<step name="finalize">
## Step 5: Finalize

- Add version number and date
- Mark sections as Draft where feedback is needed
- Include a change log placeholder
- Suggest stakeholders for review
</step>

</process>

<anti_patterns>
Avoid these common PRD mistakes:
- **Too vague**: "Make it fast" → "Page loads in < 500ms on 3G"
- **Too detailed**: Implementation specifics belong in TDD, not PRD
- **No data**: Claims without evidence ("users want this")
- **Missing scope**: Undefined boundaries lead to scope creep
- **Static document**: PRDs must evolve with the product
- **Excessive delegation**: Don't leave all decisions to designers/engineers
</anti_patterns>

<success_criteria>
A complete PRD:
- [ ] Clearly defines the problem being solved
- [ ] Has measurable success metrics
- [ ] Includes prioritized requirements with acceptance criteria
- [ ] Defines scope (in and out)
- [ ] Contains user personas or user stories
- [ ] Has navigable structure with clear headers
- [ ] Is readable by all stakeholders (PM, eng, design, exec)
- [ ] Includes version and change tracking
</success_criteria>
