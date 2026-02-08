<required_reading>
**Read these reference files NOW:**
1. references/database-properties.md
2. references/mcp-tools.md
</required_reading>

<process>

**Step 1: Gather Database Requirements**

Ask the user (if not already clear):
- What will this database track? (projects, tasks, contacts, inventory, etc.)
- What properties/columns do you need?
- What view would you prefer? (table, board, calendar, list, gallery)
- Where should it live? (parent page URL, or private)

**Step 2: Design the Schema**

Based on the use case, design the database properties. Reference `references/database-properties.md` for available property types.

Common database patterns:
- **Task Tracker:** Name, Status (select), Priority (select), Assignee (person), Due Date (date), Tags (multi-select)
- **Project Board:** Name, Status (select), Owner (person), Start/End Date, Progress (number), Description (rich text)
- **Content Calendar:** Name, Status (select), Publish Date (date), Author (person), Channel (multi-select), Type (select)
- **CRM/Contacts:** Name, Company (text), Email (email), Phone (phone), Status (select), Last Contact (date), Notes (rich text)
- **Meeting Log:** Name, Date (date), Attendees (person), Type (select), Action Items (rich text), Recording (URL)

**Step 3: Create the Database**

Use `notion-create-database` MCP tool with:
- Title/name for the database
- Property definitions with types
- Parent location
- Initial view configuration

**Step 4: Add Initial Data (Optional)**

If the user wants seed data:
- Use `notion-create-pages` to batch-create database entries
- Set properties for each entry

**Step 5: Verify**

- Use `notion-fetch` on the database URL to verify structure
- Confirm properties and views are correct
- Share the URL with the user
</process>

<success_criteria>
Database creation is complete when:
- [ ] Database created with correct properties and types
- [ ] Placed in the correct parent location
- [ ] View configured appropriately for the use case
- [ ] Initial data added if requested
- [ ] Database URL shared with user
</success_criteria>
