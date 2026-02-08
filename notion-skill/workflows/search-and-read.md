<required_reading>
**Read this reference file NOW:**
1. references/mcp-tools.md
</required_reading>

<process>

**Step 1: Determine Search Intent**

If the user has already provided a URL:
- Skip search, go directly to Step 3 (Fetch)

If the user needs to find something:
- Clarify what they're looking for if vague
- Determine if this is a keyword search or semantic search

**Step 2: Search the Workspace**

Use `notion-search` MCP tool:
- Pass the search query
- Note: Semantic search across connected apps (Slack, Drive, Jira) requires Notion AI plan
- Without Notion AI, search is limited to Notion workspace content
- Rate limit: 30 searches/minute

If results are too broad, help the user refine the query.

**Step 3: Fetch Page Content**

Use `notion-fetch` MCP tool with the page or database URL:
- Returns the full content of the page
- For databases, returns the schema and entries

**Step 4: Present Results**

Present the content to the user in a clear, organized way:
- Summarize long pages
- Highlight the parts relevant to their query
- Offer to take action (update, duplicate, move) if appropriate
</process>

<success_criteria>
Search/read is complete when:
- [ ] User's target content found and presented
- [ ] Content is clearly summarized if lengthy
- [ ] Follow-up actions offered if relevant
</success_criteria>
