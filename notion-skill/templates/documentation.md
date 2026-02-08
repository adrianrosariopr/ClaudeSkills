<template_info>
**Template:** Technical Documentation
**Required inputs:** Topic/title, subject area
**Optional inputs:** Code examples, API endpoints, prerequisites
</template_info>

<template_content>
```markdown
> [!NOTE]
> **Last Updated:** {date} | **Owner:** {owner}
> **Status:** Draft / Published / Needs Review

---

# {Title}

{1-2 sentence overview of what this document covers and who it's for.}

---

# Prerequisites

- {prerequisite_1}
- {prerequisite_2}
- {prerequisite_3}

# Getting Started

## Installation

```bash
{installation_command}
```

## Configuration

```{language}
{configuration_example}
```

> [!TIP]
> {helpful tip about configuration}

---

# Usage

## Basic Example

```{language}
{basic_code_example}
```

## Advanced Example

```{language}
{advanced_code_example}
```

---

# API Reference

| Endpoint / Method | Description | Parameters |
|-------------------|-------------|------------|
| `{endpoint_1}` | {description} | {params} |
| `{endpoint_2}` | {description} | {params} |
| `{endpoint_3}` | {description} | {params} |

<details>
<summary>Detailed Parameter Reference</summary>

### {endpoint_1}

{detailed description, request/response examples}

### {endpoint_2}

{detailed description, request/response examples}

</details>

---

# Troubleshooting

<details>
<summary>Common Issues</summary>

**Problem:** {issue_1}
**Solution:** {solution_1}

**Problem:** {issue_2}
**Solution:** {solution_2}

</details>

---

# FAQ

<details>
<summary>How do I...?</summary>
{answer}
</details>

<details>
<summary>What happens when...?</summary>
{answer}
</details>

---

# Changelog

| Date | Change | Author |
|------|--------|--------|
| {date} | Initial draft | {author} |
```
</template_content>
