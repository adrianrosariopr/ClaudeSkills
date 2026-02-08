<required_reading>
**Read these reference files NOW:**
1. references/block-types.md
2. references/design-patterns.md
</required_reading>

<process>

**Step 1: Gather Page Details**

Ask the user (if not already provided):
- What is this page about? (topic/purpose)
- Where should it live? (parent page URL, or create as private page)
- Any specific sections or structure in mind?

If the user has given clear intent (e.g., "create a project overview page for Project X"), skip unnecessary questions and proceed.

**Step 2: Design the Page Structure**

Plan the page layout before creating. A beautiful Notion page uses:

- **Icon + Cover concept** (mention to user - they'll need to add manually since MCP can't upload images)
- **Clear heading hierarchy** (H1 for title sections, H2 for subsections, H3 for details)
- **Visual variety** - Mix block types: callouts for key info, toggles for details, quotes for emphasis, dividers between sections
- **White space** - Empty paragraphs between major sections for breathing room

**Step 3: Compose the Content**

Write the page content in Notion-flavored Markdown. Use the block types from `references/block-types.md` and design patterns from `references/design-patterns.md`.

Structure the content with:
1. A compelling opening (callout or quote with context)
2. Clear sections with headings
3. Visual elements interspersed (callouts, toggles, dividers)
4. Actionable items (to-do lists, tables) where appropriate
5. A strong closing section

**Step 4: Create the Page**

Use the `notion-create-pages` MCP tool:
- Set the parent page/database if specified
- Pass the Markdown content
- Set any properties (for database pages)

**Step 5: Verify and Share**

- Use `notion-fetch` to verify the page was created correctly
- Share the page URL with the user
- Suggest enhancements they can add manually (cover image, icon, embedded content)
</process>

<success_criteria>
Page creation is complete when:
- [ ] Page created in the correct location
- [ ] Content uses 3+ different block types (not just paragraphs)
- [ ] Clear heading hierarchy established
- [ ] Visual elements (callouts, dividers, toggles) included
- [ ] Page URL shared with user
</success_criteria>
