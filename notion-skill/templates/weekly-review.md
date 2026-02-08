<template_info>
**Template:** Weekly Review
**Required inputs:** Week dates, name/team
**Optional inputs:** Goals from last week, metrics
</template_info>

<template_content>
```markdown
> [!NOTE]
> **Week of:** {start_date} - {end_date}
> **Author:** {name}

---

# Wins

- {win_1}
- {win_2}
- {win_3}

> [!TIP]
> **Highlight of the Week:** {standout_achievement}

---

# Challenges

- {challenge_1}
- {challenge_2}

# Lessons Learned

> {key_insight_or_reflection}

---

# Metrics

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| {metric_1} | {target} | {actual} | On Track / Behind / Ahead |
| {metric_2} | {target} | {actual} | On Track / Behind / Ahead |
| {metric_3} | {target} | {actual} | On Track / Behind / Ahead |

---

# Last Week's Goals: Review

- [x] {completed_goal_1}
- [x] {completed_goal_2}
- [ ] {incomplete_goal} - *Carried over to next week*

# Next Week's Goals

1. {goal_1}
2. {goal_2}
3. {goal_3}

---

<details>
<summary>Notes & Misc</summary>

- {note_1}
- {note_2}

</details>
```
</template_content>
