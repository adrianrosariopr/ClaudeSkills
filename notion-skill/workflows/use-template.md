<required_reading>
**Read the appropriate template from `templates/` based on user's request:**
- Meeting notes -> templates/meeting-notes.md
- Project brief -> templates/project-brief.md
- Documentation -> templates/documentation.md
- Weekly review -> templates/weekly-review.md
- Feature spec -> templates/feature-spec.md

Also read:
1. references/design-patterns.md
</required_reading>

<process>

**Step 1: Select Template**

If the user hasn't specified a template type, present the options:

| Template | Best For |
|----------|----------|
| **Meeting Notes** | Stand-ups, 1:1s, team meetings, client calls |
| **Project Brief** | New project kickoffs, proposals, overviews |
| **Documentation** | Technical docs, guides, how-tos, runbooks |
| **Weekly Review** | Personal/team weekly reflections and planning |
| **Feature Spec** | Product requirements, feature proposals, PRDs |

Wait for selection before proceeding.

**Step 2: Gather Context**

Based on the chosen template, ask for the specific details needed to fill it in. Each template file lists its required inputs.

Adapt the questions to what the user has already provided - skip what's obvious.

**Step 3: Fill the Template**

Read the selected template from `templates/` and fill it with the user's information:
- Replace all placeholder content with real data
- Maintain the visual structure and formatting
- Add or remove sections based on relevance

**Step 4: Create the Page**

Use `notion-create-pages` MCP tool:
- Set parent location if specified
- Pass the filled template as Markdown content

**Step 5: Share and Suggest**

- Share the page URL
- Suggest customizations: "You might want to add a cover image, change the icon, or adjust the color scheme"
- Offer to create related pages (e.g., sub-pages for meeting action items)
</process>

<success_criteria>
Template page is complete when:
- [ ] Correct template selected and customized
- [ ] All placeholder content replaced with real information
- [ ] Page created in correct location
- [ ] Visual formatting preserved from template
- [ ] URL shared with user
</success_criteria>
