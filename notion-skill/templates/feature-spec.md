<template_info>
**Template:** Feature Specification
**Required inputs:** Feature name, problem statement, proposed solution
**Optional inputs:** User stories, acceptance criteria, technical details, mockups
</template_info>

<template_content>
```markdown
> [!NOTE]
> **Feature:** {feature_name}
> **Status:** Draft | **Priority:** {priority}
> **Author:** {author} | **Date:** {date}

---

# Problem Statement

{Clear description of the problem this feature solves. Who is affected and how?}

> [!WARNING]
> **Impact if not addressed:** {consequence of not building this}

---

# Proposed Solution

{High-level description of the solution approach.}

## User Stories

- As a {user_type}, I want to {action} so that {benefit}
- As a {user_type}, I want to {action} so that {benefit}
- As a {user_type}, I want to {action} so that {benefit}

---

# Requirements

## Functional Requirements

| # | Requirement | Priority |
|---|-------------|----------|
| FR-1 | {requirement_1} | Must Have |
| FR-2 | {requirement_2} | Must Have |
| FR-3 | {requirement_3} | Should Have |
| FR-4 | {requirement_4} | Nice to Have |

## Non-Functional Requirements

- **Performance:** {performance_requirement}
- **Security:** {security_requirement}
- **Accessibility:** {accessibility_requirement}

---

# Acceptance Criteria

- [ ] {criteria_1}
- [ ] {criteria_2}
- [ ] {criteria_3}
- [ ] {criteria_4}

---

<details>
<summary>Technical Design</summary>

## Architecture

{Technical approach, components involved, data flow}

## API Changes

| Endpoint | Method | Description |
|----------|--------|-------------|
| {endpoint} | {method} | {description} |

## Database Changes

{Schema changes, migrations needed}

## Dependencies

- {dependency_1}
- {dependency_2}

</details>

<details>
<summary>Edge Cases & Error Handling</summary>

| Scenario | Expected Behavior |
|----------|-------------------|
| {edge_case_1} | {behavior} |
| {edge_case_2} | {behavior} |
| {edge_case_3} | {behavior} |

</details>

---

# Success Metrics

1. {metric_1} - Measured by: {method}
2. {metric_2} - Measured by: {method}

# Timeline

| Milestone | Target Date |
|-----------|-------------|
| Spec Approved | {date} |
| Development Start | {date} |
| Code Complete | {date} |
| QA Complete | {date} |
| Release | {date} |
```
</template_content>
