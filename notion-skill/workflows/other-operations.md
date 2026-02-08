<required_reading>
**Read this reference file NOW:**
1. references/mcp-tools.md
</required_reading>

<process>

**Step 1: Identify the Operation**

Determine what the user wants to do:

| Operation | MCP Tool | Notes |
|-----------|----------|-------|
| Move pages | `notion-move-pages` | Move one or more pages/databases to a new parent |
| Duplicate page | `notion-duplicate-page` | Creates an async copy |
| Add comment | `notion-create-comment` | Page-level comments only |
| Read comments | `notion-get-comments` | Lists all comments with threads |
| List teams | `notion-get-teams` | Workspace teamspaces |
| List users | `notion-get-users` | All workspace members |

**Step 2: Gather Required Info**

For each operation, confirm:
- **Move:** Source page(s) URL and destination parent URL
- **Duplicate:** Source page URL
- **Comment:** Target page URL and comment text
- **Read comments:** Target page URL
- **Teams/Users:** No additional info needed

**Step 3: Execute**

Call the appropriate MCP tool with the gathered parameters.

**Step 4: Confirm**

- Verify the operation completed successfully
- For moves: confirm new location
- For duplicates: share the new page URL (note: async, may take a moment)
- For comments: confirm comment was added
- Share results with the user
</process>

<success_criteria>
Operation is complete when:
- [ ] Correct MCP tool used for the operation
- [ ] Operation completed without errors
- [ ] Results confirmed and shared with user
</success_criteria>
