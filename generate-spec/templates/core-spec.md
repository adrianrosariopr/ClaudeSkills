# Core Spec Template

<usage>
This template documents WHAT the product IS and HOW it works for **non-technical stakeholders** like Product Managers and Designers.

**Audience:** People who are NOT engineers. Write so your non-technical manager would understand every word.

Key rules:
- NO technical jargon (no APIs, endpoints, middleware, schemas)
- Describe what users SEE and DO, not how the code works
- Use plain language: "users log in" not "authentication flow"
- Focus on user value and experience, not implementation
</usage>

<template>
```markdown
---
title: "{{PROJECT_NAME}} Spec"
type: project-spec
project: "{{project-slug}}"
project_type: "{{website|webapp|mobile|game}}"
created: "{{YYYY-MM-DD}}"
updated: "{{YYYY-MM-DD}}"
tags:
  - spec
  - {{project-type}}
  - {{primary-framework}}
---

# {{PROJECT_NAME}}

## Overview
{{2-3 paragraphs explaining what this product is and how it works at a high level. Write as if explaining to someone who has never seen it. Be specific about the core mechanics/functionality.}}

## Goals
{{Brief bullet list of why this product exists - what problem it solves, what it's trying to achieve. Keep it short.}}

- {{Goal 1}}
- {{Goal 2}}
- {{Goal 3}}

## Who Is It For?
{{Simple description of target users. No complex persona tables - just who uses this and why.}}

## User Experience Highlights
{{Quick bullet summary of key UX elements - what the user sees and does. Think lobby, main screens, key interactions.}}

- **{{Area 1}}:** {{Brief description}}
- **{{Area 2}}:** {{Brief description}}
- **{{Area 3}}:** {{Brief description}}

---

## Feature Breakdown

### {{Feature/Screen 1}}

{{Description of what this feature/screen is and what it does.}}

**Key Elements:**
- {{Element 1}} - {{What it does}}
- {{Element 2}} - {{What it does}}
- {{Element 3}} - {{What it does}}

{{Include screenshots or describe the UI if relevant}}

### {{Feature/Screen 2}}

{{Continue for each major feature/screen...}}

---

## Feature Flows

### {{Flow 1: e.g., User Registration}}

| Step | Description |
|------|-------------|
| 1 | {{What happens first}} |
| 2 | {{What happens next}} |
| 3 | {{Continue...}} |

### {{Flow 2: e.g., Making a Purchase}}

| Step | Description |
|------|-------------|
| 1 | {{Step description}} |
| 2 | {{Step description}} |

{{Continue for each major user flow...}}

---

## Core Systems

### Internal Systems
{{List the internal systems, services, or modules this product uses}}

- {{System 1}} - {{What it does}}
- {{System 2}} - {{What it does}}

### External Integrations
{{List third-party services and what they're used for}}

- {{Service 1}} - {{What it does}}
- {{Service 2}} - {{What it does}}

---

## Built With

| What | Tool |
|------|------|
| User Interface | {{e.g., React - a popular interface builder}} |
| Server | {{e.g., Laravel - handles business logic}} |
| Data Storage | {{e.g., SQLite - stores user information}} |
| Hosting | {{e.g., AWS - where the app runs}} |

*Note: This section is for reference only - no technical knowledge needed.*

---

## Change Log

| Date | Change |
|------|--------|
| {{YYYY-MM-DD}} | {{Description of change}} |

---

*Generated {{YYYY-MM-DD}}*
```
</template>

<guidelines>
1. **Write for non-technical readers** - Product Managers and Designers, not engineers
2. **Plain language only** - No jargon. If a term needs explaining, replace it
3. **Describe, don't prescribe** - Write "The user clicks X" not "The system shall allow X"
4. **Focus on user experience** - What users see, click, and feel
5. **Use tables for flows** - Step-by-step tables are easy to scan
6. **Keep it practical** - What does someone need to understand this product?
7. **No code or technical details** - Implementation is irrelevant to this audience
8. **Screenshots welcome** - A picture is worth a thousand words for non-technical readers
</guidelines>
