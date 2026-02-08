---
name: notion-skill
description: Create, read, update, and manage beautiful Notion pages and databases via the Notion MCP server. Use when working with Notion, creating pages, managing databases, or designing rich content layouts.
---

<essential_principles>

**Notion MCP Connection:** This skill requires the Notion MCP server. Set up with:
```
claude mcp add --transport http notion https://mcp.notion.com/mcp
```
Then authenticate via OAuth when prompted. All operations go through MCP tools - never call the Notion REST API directly.

**Content Format:** The Notion MCP `notion-create-pages` and `notion-update-page` tools accept Notion-flavored Markdown for content. Use Markdown formatting (headings, lists, bold, code blocks, etc.) rather than raw JSON block objects.

**MCP Tools Available:**

| Tool | Purpose |
|------|---------|
| `notion-search` | Search workspace (and connected apps with Notion AI) |
| `notion-fetch` | Read a page or database by URL |
| `notion-create-pages` | Create one or more pages with content |
| `notion-update-page` | Update page properties or content |
| `notion-move-pages` | Move pages/databases to new parent |
| `notion-duplicate-page` | Duplicate a page (async) |
| `notion-create-database` | Create database with properties and views |
| `notion-update-data-source` | Update data source properties |
| `notion-create-comment` | Add comments to pages |
| `notion-get-comments` | List comments on a page |
| `notion-get-teams` | List workspace teams |
| `notion-get-users` | List workspace users |

**Design-First Approach:** When creating pages, always think about visual hierarchy and readability. Use callouts for emphasis, toggles for collapsible sections, dividers for separation, and varied block types to create engaging layouts. Never create walls of plain text.

**Rate Limits:** 180 requests/minute average (3/second). `notion-search` is limited to 30/minute. Batch operations when possible using `notion-create-pages` (supports multiple pages per call).

**Limitations:**
- File/image uploads not supported via MCP (use external URLs instead)
- Block-level comments not supported (page-level only)
- `notion-query-data-sources` requires Enterprise + Notion AI
- `notion-query-database-view` requires Business+ with Notion AI
</essential_principles>

<intake>
What would you like to do in Notion?

1. **Create a page** - Build a beautiful new page with rich content
2. **Create a database** - Set up a new database with properties and views
3. **Read/search** - Find and read existing pages or databases
4. **Update content** - Modify an existing page or database
5. **Use a template** - Create a page from a pre-built beautiful template
6. **Something else** - Move, duplicate, comment, or other operations

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Next Action | Workflow |
|----------|-------------|----------|
| 1, "create", "page", "new page", "build" | Gather page details | `workflows/create-page.md` |
| 2, "database", "table", "board", "db" | Gather database details | `workflows/manage-database.md` |
| 3, "search", "find", "read", "fetch", "look up" | Execute search/fetch | `workflows/search-and-read.md` |
| 4, "update", "edit", "modify", "change" | Identify target and changes | `workflows/update-content.md` |
| 5, "template", "meeting notes", "project brief", "wiki", "docs" | Show template options | `workflows/use-template.md` |
| 6, "move", "duplicate", "comment", "other" | Clarify and execute | `workflows/other-operations.md` |

**Intent-based routing (if user provides clear intent without selecting menu):**
- "create a meeting notes page" -> `workflows/use-template.md`
- "make a project tracker database" -> `workflows/manage-database.md`
- "find my Q1 planning doc" -> `workflows/search-and-read.md`
- "add a section to [page]" -> `workflows/update-content.md`
- "build me a beautiful landing page for..." -> `workflows/create-page.md`

**After reading the workflow, follow it exactly.**
</routing>

<reference_index>
All in `references/`:

**MCP Tools:** mcp-tools.md - Detailed tool parameters and usage patterns
**Block Types:** block-types.md - All Notion block types with Markdown syntax
**Design Patterns:** design-patterns.md - Beautiful layout recipes and formatting techniques
**Database Properties:** database-properties.md - Property types, views, and configuration
</reference_index>

<templates_index>
All in `templates/`:

| Template | Purpose |
|----------|---------|
| meeting-notes.md | Structured meeting notes with attendees, agenda, actions |
| project-brief.md | Project overview with goals, timeline, resources |
| documentation.md | Technical documentation with sections and code blocks |
| weekly-review.md | Weekly review with wins, challenges, next steps |
| feature-spec.md | Feature specification with requirements and acceptance criteria |
</templates_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| create-page.md | Build beautiful pages with rich content from scratch |
| manage-database.md | Create and configure databases with properties and views |
| search-and-read.md | Search workspace and read page content |
| update-content.md | Modify existing pages, add sections, change properties |
| use-template.md | Create pages from pre-built beautiful templates |
| other-operations.md | Move, duplicate, comment, and utility operations |
</workflows_index>

<success_criteria>
A successful Notion operation:
- Completes without MCP tool errors
- Creates visually appealing content with varied block types
- Uses proper hierarchy (headings, sections, toggles)
- Includes visual elements (callouts, dividers, icons) for engagement
- Places content in the correct parent location
- Sets appropriate properties for database entries
</success_criteria>
