<overview>
Complete reference for all Notion MCP tools available through the hosted server at `mcp.notion.com/mcp`. These tools are called via the MCP protocol - never use raw HTTP API calls.
</overview>

<tools>

<tool name="notion-search">
**Purpose:** Search across your Notion workspace and connected apps.

**Usage:** Pass a search query string. Results include pages and databases matching the query.

**Notes:**
- With Notion AI: Semantic search across Notion + connected apps (Slack, Google Drive, Jira)
- Without Notion AI: Keyword search limited to Notion workspace only
- Rate limit: 30 requests/minute (stricter than general limit)
- Use specific, targeted queries for best results
</tool>

<tool name="notion-fetch">
**Purpose:** Read the content of a Notion page or database by URL.

**Usage:** Pass the full Notion URL (e.g., `https://notion.so/Page-Title-abc123`).

**Returns:** Full page content including blocks, properties, and metadata. For databases, returns schema and entries.

**Notes:**
- Works with both page and database URLs
- Returns content in a readable format
- Use this to verify changes after create/update operations
</tool>

<tool name="notion-create-pages">
**Purpose:** Create one or more new Notion pages.

**Usage:**
- Provide content in Notion-flavored Markdown
- Optionally specify a parent page/database
- Can create multiple pages in a single call (batch)
- If no parent specified, creates a private page

**Content format:** Notion-flavored Markdown with headings, lists, bold, italic, code blocks, callouts, toggles, dividers, tables, to-dos, quotes, and more.

**Notes:**
- Batch multiple pages in one call to respect rate limits
- For database pages, set properties in addition to content
- File/image uploads not supported - use external URLs for images
</tool>

<tool name="notion-update-page">
**Purpose:** Update an existing page's properties or content.

**Usage:**
- Specify the page to update (by URL)
- Provide updated content or property changes
- Can modify status fields, add sections, replace content

**Notes:**
- Be careful to preserve existing content when adding to a page
- Read the page first with `notion-fetch` to understand current structure
</tool>

<tool name="notion-move-pages">
**Purpose:** Move one or more pages or databases to a new parent location.

**Usage:** Provide source page(s) and destination parent.

**Notes:**
- Can move multiple items at once
- Works with both pages and databases
</tool>

<tool name="notion-duplicate-page">
**Purpose:** Create a copy of an existing page.

**Usage:** Provide the source page URL.

**Notes:**
- Operation completes asynchronously - the page may not be immediately available
- Creates a full copy including all content and sub-pages
</tool>

<tool name="notion-create-database">
**Purpose:** Create a new database with properties and views.

**Usage:**
- Specify database title/name
- Define properties with their types
- Set initial view configuration (table, board, calendar, list, gallery)
- Specify parent location

**Notes:**
- See `references/database-properties.md` for available property types
- Consider the primary view type based on use case
</tool>

<tool name="notion-update-data-source">
**Purpose:** Update a data source's properties, name, description, or attributes.

**Usage:** Specify the data source and the changes to apply.
</tool>

<tool name="notion-query-data-sources">
**Purpose:** Query multiple data sources with structured summaries, grouping, and filters.

**Requirements:** Enterprise plan with Notion AI.
</tool>

<tool name="notion-query-database-view">
**Purpose:** Query database data using pre-defined view filters and sorts.

**Requirements:** Business plan or above with Notion AI.
</tool>

<tool name="notion-create-comment">
**Purpose:** Add a comment to a page.

**Usage:** Specify the page URL and comment text.

**Notes:**
- Page-level comments only (block-level comments not yet supported)
- Useful for leaving notes, feedback, or action items
</tool>

<tool name="notion-get-comments">
**Purpose:** List all comments on a page, including threaded discussions.

**Usage:** Provide the page URL.
</tool>

<tool name="notion-get-teams">
**Purpose:** Retrieve the list of teams (teamspaces) in the workspace.
</tool>

<tool name="notion-get-users">
**Purpose:** List all users in the workspace with their details.
</tool>

<tool name="notion-get-user">
**Purpose:** Retrieve information about a specific user by ID.
</tool>

<tool name="notion-get-self">
**Purpose:** Retrieve bot user info and connected workspace details.
</tool>

</tools>

<rate_limits>
- **General:** 180 requests/minute average (3 requests/second)
- **notion-search:** 30 requests/minute
- **Batch when possible:** Use `notion-create-pages` for multiple pages in one call
</rate_limits>
