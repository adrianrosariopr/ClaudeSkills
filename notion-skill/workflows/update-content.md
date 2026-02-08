<required_reading>
**Read these reference files NOW:**
1. references/block-types.md
2. references/mcp-tools.md
</required_reading>

<process>

**Step 1: Identify the Target**

Ask the user (if not already clear):
- Which page needs updating? (URL or search for it)
- What changes are needed? (add content, modify sections, update properties)

If no URL provided, use `notion-search` to find the page.

**Step 2: Read Current Content**

Use `notion-fetch` to read the current page content:
- Understand the existing structure
- Identify where changes should go
- Note current properties (for database pages)

**Step 3: Plan the Changes**

Describe what will be changed before executing:
- New sections being added
- Content being modified
- Properties being updated

Get user confirmation for significant changes.

**Step 4: Apply Updates**

Use `notion-update-page` MCP tool:
- For content changes: provide updated Markdown content
- For property changes: specify property values
- Maintain existing content structure when adding to a page

**Step 5: Verify Changes**

- Use `notion-fetch` to verify the update applied correctly
- Share confirmation with the user
- Note any sections that may need manual adjustment
</process>

<success_criteria>
Update is complete when:
- [ ] Target page identified correctly
- [ ] Changes applied without losing existing content
- [ ] User confirmed the changes look correct
- [ ] Visual quality maintained or improved
</success_criteria>
