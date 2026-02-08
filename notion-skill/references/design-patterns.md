<overview>
Recipes and patterns for creating visually appealing, well-structured Notion pages. Apply these patterns when building pages to ensure they look professional and are easy to navigate.
</overview>

<core_principles>

**Visual Hierarchy** - Guide the reader's eye through clear structure:
- H1 for major sections, H2 for subsections, H3 for details
- Never skip heading levels (H1 -> H3 without H2)
- Use callouts to draw attention to key information
- Use dividers to separate major sections

**Block Variety** - Avoid walls of plain text:
- Mix at least 3-4 different block types per page
- Alternate between text, visual elements, and interactive blocks
- Use callouts, quotes, toggles, and tables to break up content

**White Space** - Let the page breathe:
- Add empty paragraphs between major sections
- Don't cram too many blocks together
- Use dividers to create clear visual breaks

**Scanability** - Make content easy to scan:
- Bold key terms and important phrases
- Use bullet points for lists of 3+ items
- Front-load important information in each section
- Keep paragraphs to 2-3 sentences max

</core_principles>

<page_layouts>

<pattern name="hero-opener">
**Hero Opener** - Start with impact

Start pages with a callout or quote that sets context:
```markdown
> [!NOTE]
> This document outlines the Q1 2026 product strategy. Last updated: Feb 2026.

---

# Product Strategy: Q1 2026

Brief overview paragraph here...
```
</pattern>

<pattern name="section-with-callout">
**Section with Callout Accent** - Key info highlighted

```markdown
## Timeline

> [!TIP]
> Key dates: Kickoff Feb 10, MVP March 15, Launch April 1

### Phase 1: Research (Feb 10-21)
- User interviews
- Competitive analysis
- Technical feasibility

### Phase 2: Build (Feb 24 - Mar 15)
- Core feature development
- Integration testing
```
</pattern>

<pattern name="toggle-sections">
**Toggle Sections** - Collapsible detail areas

Use toggles for content that's important but doesn't need to be visible by default:
```markdown
## API Documentation

<details>
<summary>Authentication</summary>

All requests require a Bearer token in the Authorization header...

</details>

<details>
<summary>Endpoints</summary>

### GET /api/users
Returns a list of all users...

</details>
```
</pattern>

<pattern name="status-dashboard">
**Status Dashboard** - Quick overview with table

```markdown
> [!NOTE]
> Project Status: **On Track**

| Area | Status | Owner | Due |
|------|--------|-------|-----|
| Backend API | In Progress | @Alex | Feb 15 |
| Frontend UI | Not Started | @Sam | Feb 22 |
| QA Testing | Blocked | @Jordan | Mar 1 |

---

## Details
```
</pattern>

<pattern name="two-column-comparison">
**Comparison Layout** - Side-by-side options via table

```markdown
## Option Comparison

| | Option A: Build | Option B: Buy |
|---|---|---|
| **Cost** | $50k development | $200/mo SaaS |
| **Timeline** | 3 months | 1 week |
| **Flexibility** | Full control | Limited customization |
| **Maintenance** | Ongoing team effort | Vendor managed |

> [!TIP]
> **Recommendation:** Option A for long-term value if budget allows.
```
</pattern>

<pattern name="checklist-section">
**Checklist Section** - Trackable action items

```markdown
## Launch Checklist

- [ ] Code review completed
- [ ] Staging environment tested
- [ ] Documentation updated
- [ ] Stakeholders notified
- [x] Feature flags configured
- [x] Monitoring dashboards set up
```
</pattern>

<pattern name="structured-closing">
**Structured Closing** - End with clear next steps

```markdown
---

## Next Steps

1. **Immediate** - Review this document and leave comments by Friday
2. **This week** - Schedule kickoff meeting with all stakeholders
3. **Next week** - Begin Phase 1 execution

> [!NOTE]
> Questions? Reach out to @ProjectLead in the #project-channel.
```
</pattern>

</page_layouts>

<visual_recipes>

<recipe name="meeting-flow">
**Meeting Page Flow:**
1. Callout with meeting metadata (date, attendees, type)
2. Divider
3. H2: Agenda (numbered list)
4. H2: Discussion Notes (paragraphs with bold key decisions)
5. H2: Action Items (to-do list with owners)
6. Divider
7. Toggle: Reference Materials (links, docs)
</recipe>

<recipe name="documentation-flow">
**Documentation Page Flow:**
1. Callout with doc metadata (last updated, owner, status)
2. Table of Contents
3. Divider
4. H1: Overview (1-2 paragraphs)
5. H1: Getting Started (code block + steps)
6. H1: Usage (examples with code blocks)
7. H1: API Reference (table + toggle details)
8. Divider
9. H2: FAQ (toggle for each question)
10. H2: Changelog (list)
</recipe>

<recipe name="project-flow">
**Project Brief Flow:**
1. Callout with project status and key dates
2. Divider
3. H1: Overview (2-3 sentences)
4. H1: Goals (bulleted list)
5. H1: Scope (table with In/Out columns)
6. H1: Timeline (table with phases/dates)
7. H1: Team (table with roles/people)
8. Divider
9. H1: Risks and Mitigations (toggle details)
10. H1: Success Metrics (numbered list)
</recipe>

</visual_recipes>

<anti_patterns>

<pitfall name="wall-of-text">
**Wall of Text** - Long paragraphs with no visual breaks. Always break up with headings, callouts, or lists.
</pitfall>

<pitfall name="flat-structure">
**Flat Structure** - All content at the same level. Use heading hierarchy to create depth.
</pitfall>

<pitfall name="no-visual-elements">
**No Visual Elements** - Only paragraphs and headings. Add callouts, tables, dividers, and toggles.
</pitfall>

<pitfall name="over-decoration">
**Over-Decoration** - Too many callouts, colors, and icons. Keep it balanced - 2-3 callouts per page is usually enough.
</pitfall>

<pitfall name="missing-context">
**Missing Context** - Jumping into content without setting the scene. Always start with a brief callout or intro paragraph.
</pitfall>

</anti_patterns>
