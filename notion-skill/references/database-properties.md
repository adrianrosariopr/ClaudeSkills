<overview>
Notion database property types, their uses, and recommended configurations for common database patterns.
</overview>

<property_types>

| Property Type | Description | Best For |
|--------------|-------------|----------|
| **Title** | Primary name field (every database has one) | Item names, page titles |
| **Rich Text** | Multi-line formatted text | Descriptions, notes, details |
| **Number** | Numeric values with optional format | Prices, quantities, scores, progress % |
| **Select** | Single choice from predefined options | Status, priority, type, category |
| **Multi-Select** | Multiple choices from predefined options | Tags, labels, skills, channels |
| **Date** | Date or date range | Due dates, start/end dates, milestones |
| **Person** | Workspace member reference | Assignee, owner, reviewer |
| **Checkbox** | Boolean true/false | Flags, completion status, yes/no fields |
| **URL** | Web link | External resources, documentation links |
| **Email** | Email address | Contact emails |
| **Phone** | Phone number | Contact numbers |
| **Formula** | Computed value from other properties | Calculated fields, conditional text |
| **Relation** | Link to another database | Cross-references, parent-child relationships |
| **Rollup** | Aggregated data from relations | Sums, counts, averages from related items |
| **Created Time** | Auto timestamp of creation | Audit trail |
| **Last Edited Time** | Auto timestamp of last edit | Freshness tracking |
| **Created By** | Auto user who created | Audit trail |
| **Last Edited By** | Auto user who last edited | Audit trail |
| **Files & Media** | Attached files | Documents, images (note: MCP can't upload) |
| **Status** | Built-in status with groups (To Do, In Progress, Done) | Workflow state tracking |

</property_types>

<view_types>

| View | Best For | Key Settings |
|------|----------|--------------|
| **Table** | Data-heavy content, spreadsheet-like | Column visibility, sort, filter |
| **Board** | Kanban workflows, visual status tracking | Group by select/status property |
| **Calendar** | Time-based content, scheduling | Date property for positioning |
| **List** | Simple text-focused lists | Minimal properties shown |
| **Gallery** | Visual content, card-based browsing | Cover image, preview properties |
| **Timeline** | Project planning, Gantt-like views | Start/end date properties |

</view_types>

<common_schemas>

<schema name="task-tracker">
**Task Tracker**
- Title: Task name
- Status: `Not Started`, `In Progress`, `In Review`, `Done` (Status type)
- Priority: `Urgent`, `High`, `Medium`, `Low` (Select)
- Assignee: (Person)
- Due Date: (Date)
- Tags: `Bug`, `Feature`, `Improvement`, `Documentation` (Multi-Select)
- Effort: `XS`, `S`, `M`, `L`, `XL` (Select)

**Recommended view:** Board grouped by Status
</schema>

<schema name="project-tracker">
**Project Tracker**
- Title: Project name
- Status: `Planning`, `Active`, `On Hold`, `Completed`, `Cancelled` (Select)
- Owner: (Person)
- Start Date: (Date)
- End Date: (Date)
- Progress: 0-100 (Number, percent format)
- Team: (Multi-Select or Relation to people DB)
- Description: (Rich Text)
- Priority: `P0`, `P1`, `P2`, `P3` (Select)

**Recommended view:** Table with Status filter, or Board grouped by Status
</schema>

<schema name="content-calendar">
**Content Calendar**
- Title: Content title
- Status: `Idea`, `Drafting`, `Review`, `Scheduled`, `Published` (Select)
- Publish Date: (Date)
- Author: (Person)
- Channel: `Blog`, `Twitter`, `LinkedIn`, `Newsletter`, `YouTube` (Multi-Select)
- Type: `Article`, `Video`, `Thread`, `Short`, `Newsletter` (Select)
- URL: Published link (URL)
- Notes: (Rich Text)

**Recommended view:** Calendar by Publish Date, with Board by Status as secondary
</schema>

<schema name="crm">
**Simple CRM**
- Title: Contact/Company name
- Email: (Email)
- Phone: (Phone)
- Company: (Rich Text)
- Status: `Lead`, `Contacted`, `Qualified`, `Proposal`, `Won`, `Lost` (Select)
- Source: `Inbound`, `Referral`, `Cold`, `Event`, `Social` (Select)
- Last Contact: (Date)
- Deal Value: (Number, currency format)
- Notes: (Rich Text)
- Website: (URL)

**Recommended view:** Board grouped by Status, with Table as secondary
</schema>

<schema name="knowledge-base">
**Knowledge Base / Wiki**
- Title: Article title
- Category: `Engineering`, `Design`, `Product`, `Operations`, `HR` (Select)
- Tags: (Multi-Select)
- Owner: (Person)
- Last Reviewed: (Date)
- Status: `Draft`, `Published`, `Needs Update`, `Archived` (Select)

**Recommended view:** List grouped by Category
</schema>

</common_schemas>
