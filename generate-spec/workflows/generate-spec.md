# Workflow: Generate Project Spec

<required_reading>
**Read these reference files NOW:**
1. references/project-detection.md
2. references/analysis-checklist.md
3. references/spec-best-practices.md
</required_reading>

<process>

## Step 1: Notion Configuration

**Notion API settings (from SKILL.md):**
| Setting | Value |
|---------|-------|
| API Token | Read from `~/.claude/.env` file (look for `NOTION_API_TOKEN=`) |
| API Version | `2022-06-28` |
| Default Parent Page | SciPlay (`d841d8f6-efb9-412a-b025-8db02c06ccc8`) |

Ask the user which Notion parent page to use if not obvious. Use the Notion search API (`POST /v1/search`) to list accessible pages if needed.

## Step 2: Project Discovery

Gather high-level project information:

```bash
# Get project name from directory
basename "$(pwd)"

# Check for common project files
ls -la | head -30

# Check git status if available
git remote -v 2>/dev/null || echo "Not a git repo"
```

## Step 3: Comprehensive Analysis

Use the Task tool with `subagent_type=Explore` to perform deep analysis:

**Analysis areas (from analysis-checklist.md):**

1. **Structure Analysis**
   - Directory layout and organization
   - Entry points (main files, index files)
   - Build/config files (package.json, composer.json, Cargo.toml, etc.)

2. **Technology Stack**
   - Languages used
   - Frameworks (React, Laravel, Unity, etc.)
   - Key dependencies and versions

3. **Architecture Patterns**
   - MVC, MVVM, component-based, ECS
   - State management approach
   - API structure (REST, GraphQL, etc.)

4. **Data Layer**
   - Database type (SQL, NoSQL, none)
   - ORM/data access patterns
   - Schema/migrations if available

5. **Authentication & Authorization**
   - Auth mechanism (sessions, JWT, OAuth)
   - Roles/permissions system
   - Protected routes/resources

6. **External Integrations**
   - Third-party APIs
   - Payment processors
   - Analytics/tracking

7. **Testing Setup**
   - Test framework
   - Test coverage areas
   - CI/CD configuration

8. **Key Features**
   - Main user-facing functionality
   - Core business logic location
   - Notable patterns or conventions

## Step 4: Auto-Detect Project Type

Based on analysis, determine project type:

**Website indicators:**
- Static site generators (Hugo, Jekyll, Astro)
- CMS (WordPress, Drupal, Strapi)
- Marketing/content focus
- No authentication system
- Minimal JavaScript interactivity

**Webapp indicators:**
- Authentication system present
- Database with user data
- CRUD operations
- SaaS patterns (subscriptions, billing)
- Rich client-side state
- API endpoints

**Mobile indicators:**
- React Native, Flutter, Swift, Kotlin
- `.xcodeproj`, `.xcworkspace` files
- `android/` directory with manifests
- Mobile-specific dependencies

**Game indicators:**
- Unity (`.unity`, `Assets/`, `ProjectSettings/`)
- Unreal (`.uproject`, `Source/`, `Content/`)
- Godot (`.godot`, `*.tscn`)
- Game-specific patterns (game loop, sprites, physics)

## Step 5: Confirm with User

Present detected type and ask for confirmation:

```
Detected project type: [TYPE]

Evidence:
- [Key indicator 1]
- [Key indicator 2]
- [Key indicator 3]

Is this correct? (yes/override to: website|webapp|mobile|game)
```

Wait for user confirmation before proceeding.

## Step 6: Generate Core Spec

Read `templates/core-spec.md` and fill each section based on analysis:

**Document Control:**
- Title: `{Project Name} Specification`
- Generated: Current date
- Version: 1.0 (auto-generated)
- Source: Link to repo if available

**Executive Summary:**
- Problem statement (infer from README, code purpose)
- Goals (what the project achieves)
- Non-goals (what it explicitly doesn't do)

**Users and Context:**
- Target users (from code, UI, README)
- Key user journeys (from routes, controllers, features)
- Constraints (tech stack limits, dependencies)

**Scope:**
- In-scope features (current functionality)
- Out-of-scope (TODOs, commented features)
- Milestones (if git tags/releases exist)

**Requirements:**
- FR-XXX: Enumerate features found
- NFRs: Performance, security, accessibility observations

**UX/UI:**
- Navigation model (from routes/pages)
- Screen/page list
- Component patterns

**Data & Integrations:**
- Data model (from schema/migrations)
- APIs (internal and external)
- Third-party services

**Release & Operations:**
- Current deployment (from configs)
- Environment setup
- Known issues/risks

## Step 7: Generate Type Addendum

Based on confirmed type, read and fill the appropriate addendum:

| Type | Template |
|------|----------|
| Website | templates/website-addendum.md |
| Webapp | templates/webapp-addendum.md |
| Mobile | templates/mobile-addendum.md |
| Game | templates/game-addendum.md |

Fill addendum sections with type-specific findings from analysis.

## Step 8: Publish to Notion

Use the Notion API to create pages. Write a Python script (or use curl via Bash) that:

1. **Creates the main spec page** under the parent page using `POST https://api.notion.com/v1/pages`
2. **Appends content blocks** using `PATCH https://api.notion.com/v1/blocks/{page_id}/children`

**Notion block types to use:**
- `heading_1`, `heading_2`, `heading_3` for headings
- `paragraph` for body text (with `rich_text` annotations for bold/italic/code)
- `bulleted_list_item` for bullet points
- `table` with `table_row` children for tables
- `divider` for section breaks
- `code` for code blocks
- `callout` for warnings/notes

**Important API limits:**
- Max 100 blocks per request — batch larger specs into multiple append calls
- Rich text content max 2000 characters per block

**Page properties:**
```json
{
  "parent": {"page_id": "{parent_page_id}"},
  "icon": {"type": "emoji", "emoji": "📊"},
  "properties": {
    "title": [{"text": {"content": "{Project Name} — Spec"}}]
  },
  "children": [... first 100 blocks ...]
}
```

## Step 9: Publish ENV Child Page

If a `.env` file exists in the project root:

1. Read the `.env` file
2. Create a child page under the spec page:

```json
{
  "parent": {"page_id": "{spec_page_id}"},
  "icon": {"type": "emoji", "emoji": "🔐"},
  "properties": {
    "title": [{"text": {"content": "ENV — Environment Variables"}}]
  },
  "children": [
    heading_2("Environment Variables"),
    paragraph("Copy everything below into your .env file."),
    divider(),
    code_block(env_contents, "plain text"),
    divider(),
    callout("This file contains secrets. Do not share publicly.", "⚠️", "yellow_background")
  ]
}
```

If no `.env` file exists, skip this step.

## Step 10: Process Feature Specs

Check if the project has feature specs in `docs/features/`:

```bash
ls docs/features/FEATURE-*.md 2>/dev/null || echo "No feature specs found"
```

If feature specs exist:

**1. Add Features section to the main spec page**

Append a table block listing each feature with name, status, and description.

**2. Create feature detail child pages**

For each `FEATURE-*.md` found, create a child page under the spec page with the full feature content converted to Notion blocks.

If no feature specs exist, skip this step.

## Step 11: Report Completion

Tell the user:
- Notion URLs for the spec page and ENV page
- Summary of what was documented
- Number of feature specs imported (if any)
- Any sections that need manual review

**Output structure (Notion page tree):**
```
{Parent Page}/
└── {Project Name} — Spec          # Main spec page
    ├── ENV — Environment Variables # Child page (if .env exists)
    └── {Feature Name}             # Child pages (if docs/features/ exists)
```

</process>

<success_criteria>
This workflow is complete when:
- [ ] Notion parent page identified
- [ ] Project comprehensively analyzed (8 analysis areas)
- [ ] Project type detected and confirmed by user
- [ ] Core spec generated with all applicable sections
- [ ] Type-specific addendum generated
- [ ] Features section added (if docs/features/ exists)
- [ ] Feature detail child pages created in Notion
- [ ] Main spec page published to Notion
- [ ] ENV child page published to Notion (if .env exists)
- [ ] User notified of Notion page URLs
</success_criteria>
