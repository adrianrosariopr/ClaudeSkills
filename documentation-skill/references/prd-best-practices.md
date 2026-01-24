<overview>
Product Requirements Documents (PRDs) define what a product or feature should do, who it's for, and why it matters. A PRD is a guide for everyone involved - engineering, design, QA, and stakeholders. 37% of project failures are due to lack of clearly defined requirements.
</overview>

<core_sections>
## Essential PRD Sections

<section name="overview">
**Overview/Executive Summary**
- 2-3 sentences capturing the product essence
- Sets context for the entire document
- Should be understandable by anyone in the company
</section>

<section name="problem">
**Problem Statement**
- What pain point or opportunity does this address?
- Include data or evidence supporting the problem
- Quantify the impact ("Users spend 15 minutes on task X")
</section>

<section name="goals">
**Goals and Success Metrics**
- Define measurable outcomes (not outputs)
- Use specific numbers: "Reduce task time by 50%"
- Include leading and lagging indicators
- OKR format works well here
</section>

<section name="users">
**Target Users/Personas**
- Who benefits from this product?
- What are their pain points and motivations?
- Include behavioral characteristics, not just demographics
</section>

<section name="requirements">
**Requirements**
- Functional: What the system does
- Non-functional: Performance, security, scalability
- Prioritize: Must-have / Should-have / Nice-to-have (MoSCoW)
- Include acceptance criteria for each
</section>

<section name="user-stories">
**User Stories**
Format: "As a [user type], I want [goal] so that [benefit]"
- Focus on user value, not implementation
- Include edge cases and error states
</section>

<section name="scope">
**Scope**
- **In scope**: What this PRD covers
- **Out of scope**: What it explicitly does NOT cover
- Critical for preventing scope creep
</section>

<section name="design">
**Design & UX**
- Link to mockups, wireframes, or prototypes
- Define user flows
- Note any accessibility requirements
</section>

<section name="timeline">
**Timeline & Milestones**
- Key phases and deliverables
- Dependencies on other teams
- NOT detailed sprint planning (that's Agile's job)
</section>

<section name="risks">
**Risks & Mitigations**
- What could go wrong?
- Likelihood and impact assessment
- Mitigation strategies for each
</section>
</core_sections>

<good_vs_bad>
## Good PRD vs Bad PRD

<comparison name="requirements">
**Bad Requirement:**
"The page should load fast"

**Good Requirement:**
"Page must achieve First Contentful Paint < 1.5 seconds on 3G connection (Lighthouse mobile)"
- Specific metric
- Measurable threshold
- Clear testing method
</comparison>

<comparison name="scope">
**Bad Scope:**
"This feature will improve user experience"

**Good Scope:**
"In scope: Checkout flow redesign (cart → payment → confirmation)
Out of scope: Account creation, product browsing, post-purchase flows"
- Clear boundaries
- Specific features named
- Explicit exclusions
</comparison>

<comparison name="success-metrics">
**Bad Success Metric:**
"Users should like the new feature"

**Good Success Metric:**
"Success if:
- Task completion rate increases from 65% to 85%
- Average time-to-complete decreases from 5 min to 2 min
- NPS score for checkout flow improves by 10 points"
</comparison>

<comparison name="user-stories">
**Bad User Story:**
"User can log in"

**Good User Story:**
"As a returning customer, I want to log in with my saved credentials so that I can access my order history and saved payment methods without re-entering information"
</comparison>
</good_vs_bad>

<audience_considerations>
## Writing for Different Audiences

**65% of PRD readers only need specific sections.** Make navigation effortless.

<audience name="executives">
**Executives** read: Overview, Goals, Timeline
- Keep these sections high-level
- Focus on business impact
- Avoid technical jargon
</audience>

<audience name="engineers">
**Engineers** read: Requirements, Technical Constraints, API specs
- Be specific and quantified
- Include edge cases
- Link to TDD for implementation details
</audience>

<audience name="designers">
**Designers** read: User Personas, User Flows, Design section
- Include user context and motivations
- Define the "why" behind requirements
- Link to research if available
</audience>

<audience name="qa">
**QA** reads: Requirements, Acceptance Criteria, Edge Cases
- Every requirement needs testable criteria
- Document expected error states
- Include boundary conditions
</audience>
</audience_considerations>

<living_document>
## PRD as Living Document

PRDs must evolve with the product:

- **Version control**: Track changes with dates and authors
- **Status markers**: Draft → In Review → Approved → Deprecated
- **Change log**: Document what changed and why
- **Review cycles**: Schedule regular updates during development
- **Archive old versions**: Don't delete, mark as superseded

A stale PRD is worse than no PRD - it actively misleads the team.
</living_document>

<key_principles>
## Key Principles

1. **Problem before solution**: Define the problem clearly before requirements
2. **Quantify everything**: Replace "fast" with "< 500ms"
3. **Include the "why"**: Rationale helps teams make good decisions
4. **Separate concerns**: PRD = what and why. TDD = how.
5. **Iterate**: Start with skeleton, add detail as you learn
6. **Keep it navigable**: Clear sections, table of contents, links
</key_principles>
