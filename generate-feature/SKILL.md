---
name: generate-feature
description: Generates implementation-ready feature documentation with phased task lists. Creates a FEATURE-SPEC.md in the project that serves as the guide for building a feature. Use when planning a new feature before implementation.
---

<objective>
Create detailed feature specifications that serve as implementation guides. The output is a structured document with clear requirements, technical approach, and a phased task list that can be tracked during development.

This skill produces documentation that `generate-spec` can reference to understand what the project is building.
</objective>

<quick_start>
Tell me about the feature you want to build:
- What is it?
- What problem does it solve?
- Any technical constraints or preferences?

I'll ask clarifying questions, then generate a FEATURE-SPEC.md in your project.
</quick_start>

<process>
**Step 1: Gather Requirements**

Ask about:
- Feature name and one-line description
- Problem it solves / why it's needed
- Key user interactions or flows
- Technical constraints (frameworks, APIs, existing patterns)
- Dependencies on other features or systems

**Step 2: Define Scope**

Clarify:
- What's in scope vs out of scope
- MVP vs future enhancements
- Success criteria (how do we know it works?)

**Step 3: Break Into Phases**

Organize work into logical phases:
- **Phase 1: Foundation** - Setup, scaffolding, core structure
- **Phase 2: Core Implementation** - Main functionality
- **Phase 3: Integration** - Connect to existing systems
- **Phase 4: Polish** - Edge cases, error handling, UX refinement
- **Phase 5: Testing & Validation** - Tests, manual verification

Each phase has specific tasks with checkboxes.

**Step 4: Generate Document**

Create `FEATURE-SPEC.md` in project root (or specified location) using the template.

**Step 5: Confirm Location**

Tell user where the file was saved and how to use it.
</process>

<output_location>
Default: `{project_root}/docs/features/FEATURE-{name}.md`

If no `docs/features/` exists, create it or ask user preference.

Alternative locations:
- `{project_root}/FEATURE-SPEC.md` (single feature)
- `{project_root}/specs/{name}.md` (multiple specs)
</output_location>

<template_structure>
The generated document follows this structure:

```markdown
# Feature: {Name}

## Overview
{One paragraph describing what this feature does}

## Problem Statement
{Why this feature is needed, what problem it solves}

## Goals
- {Goal 1}
- {Goal 2}

## Non-Goals (Out of Scope)
- {Explicitly not doing X}
- {Saving Y for future}

## User Experience
{How users interact with this feature - flows, screens, actions}

## Technical Approach
{High-level technical design - what components, what patterns}

### Dependencies
- {Existing system/feature this depends on}

### New Components
- {New files/classes/modules to create}

---

## Implementation Phases

### Phase 1: Foundation
- [ ] Task 1
- [ ] Task 2

### Phase 2: Core Implementation
- [ ] Task 1
- [ ] Task 2

### Phase 3: Integration
- [ ] Task 1
- [ ] Task 2

### Phase 4: Polish
- [ ] Task 1
- [ ] Task 2

### Phase 5: Testing & Validation
- [ ] Task 1
- [ ] Task 2

---

## Success Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}

## Open Questions
- {Question that needs answering during implementation}

---

*Created: {date}*
*Status: Draft*
```
</template_structure>

<guidelines>
**Writing Good Tasks:**
- Start with verb: "Create", "Add", "Implement", "Configure"
- Be specific: "Create UserService class" not "Set up service"
- One task = one commit (roughly)
- Include file paths when known

**Phase Guidelines:**
- Foundation: No feature code yet, just setup
- Core: The main thing works in isolation
- Integration: Connects to rest of system
- Polish: Error states, loading states, edge cases
- Testing: Unit tests, integration tests, manual QA

**Keep It Practical:**
- Tasks should be completable in one sitting
- If a task is too big, split it
- Mark dependencies between tasks if order matters
</guidelines>

<success_criteria>
Feature spec is complete when:
- [ ] Feature clearly described (what it does, why)
- [ ] Scope defined (in/out)
- [ ] Technical approach outlined
- [ ] Phases have specific, actionable tasks
- [ ] Success criteria defined
- [ ] File saved to project
- [ ] User knows where to find it
</success_criteria>
