<template_info>
**Template:** Meeting Notes
**Required inputs:** Meeting title, date, attendees, meeting type
**Optional inputs:** Agenda items, pre-read links
</template_info>

<template_content>
```markdown
> [!NOTE]
> **Meeting:** {title}
> **Date:** {date} | **Type:** {type}
> **Attendees:** {attendees}

---

# Agenda

1. {agenda_item_1}
2. {agenda_item_2}
3. {agenda_item_3}

---

# Discussion Notes

## {agenda_item_1}

Key points discussed...

> **Decision:** {decision made, if any}

## {agenda_item_2}

Key points discussed...

## {agenda_item_3}

Key points discussed...

---

# Action Items

- [ ] {action_1} - **Owner:** {person} | **Due:** {date}
- [ ] {action_2} - **Owner:** {person} | **Due:** {date}
- [ ] {action_3} - **Owner:** {person} | **Due:** {date}

---

<details>
<summary>Reference Materials</summary>

- [Pre-read document](url)
- [Related project page](url)
- [Previous meeting notes](url)

</details>

<details>
<summary>Parking Lot</summary>

Items to discuss in future meetings:
- {deferred_topic_1}
- {deferred_topic_2}

</details>
```
</template_content>
