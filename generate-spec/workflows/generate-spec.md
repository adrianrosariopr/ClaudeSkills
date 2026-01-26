# Workflow: Generate Project Spec

<required_reading>
**Read these reference files NOW:**
1. references/project-detection.md
2. references/analysis-checklist.md
3. references/spec-best-practices.md
</required_reading>

<process>

## Step 1: Use Configured Obsidian Vault

**Vault paths:**
| Platform | Path |
|----------|------|
| Mac | `/Users/adrian-rosario/Library/CloudStorage/GoogleDrive-adrianrosariopr@gmail.com/My Drive/Obsidian` |
| Windows | `TODO` |

Detect platform and use the appropriate path. No need to search.

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

## Step 8: Add Obsidian Frontmatter

Prepend YAML frontmatter for Obsidian:

```yaml
---
title: "{Project Name} Specification"
type: project-spec
project: "{project-name}"
project_type: "{website|webapp|mobile|game}"
created: "{YYYY-MM-DD}"
updated: "{YYYY-MM-DD}"
status: draft
tags:
  - project-spec
  - {project-type}
  - {tech-stack-tags}
aliases:
  - "{Project Name} Spec"
  - "{project-name}-spec"
---
```

## Step 9: Save to Obsidian Vault

Create directory and save file:

```bash
# Create project folder in vault root
mkdir -p "{vault_path}/{project-name}"

# Save spec
# Use Write tool to save the complete spec
```

Save path: `{vault_path}/{project-name}/PROJECT-SPEC.md`

## Step 10: Copy Environment Variables

If a `.env` file exists in the project root, create `ENV.md` in the vault:

1. Read the project's `.env` file
2. Create `ENV.md` with this structure:

```markdown
---
title: "{Project Name} - Environment Variables"
type: env-config
project: "{project-name}"
updated: "{YYYY-MM-DD}"
---

# Environment Variables

Copy everything below the line into your `.env` file.

---

\`\`\`env
{contents of .env file}
\`\`\`
```

Save path: `{vault_path}/{project-name}/ENV.md`

If no `.env` file exists, skip this step.

## Step 11: Process Feature Specs

Check if the project has feature specs in `docs/features/`:

```bash
ls docs/features/FEATURE-*.md 2>/dev/null || echo "No feature specs found"
```

If feature specs exist:

**1. Add Features section to PROJECT-SPEC.md**

Add a section after the main content:

```markdown
---

## Features

| Feature | Status | Description |
|---------|--------|-------------|
| [[features/{feature-name}|{Feature Name}]] | {Draft/In Progress/Complete} | {One-line summary} |

See individual feature pages for implementation details and task lists.
```

**2. Create feature detail pages**

For each `FEATURE-*.md` found:

```bash
mkdir -p "{vault_path}/{project-name}/features"
```

Create `{vault_path}/{project-name}/features/{feature-name}.md` with:

```markdown
---
title: "{Feature Name}"
type: feature-spec
project: "{project-name}"
parent: "[[PROJECT-SPEC]]"
status: {from original spec}
created: "{date from original}"
updated: "{YYYY-MM-DD}"
tags:
  - feature-spec
  - {project-name}
---

{Full content from docs/features/FEATURE-*.md}

---

*Imported from project: `docs/features/FEATURE-{name}.md`*
```

**3. Extract summary for main spec**

From each feature spec, extract:
- Feature name (from `# Feature: {Name}`)
- Status (from frontmatter or bottom of file)
- One-line description (from Overview section, first sentence)

If no feature specs exist, skip this step.

## Step 12: Report Completion

Tell the user:
- Where the spec was saved
- Summary of what was documented
- Number of feature specs imported (if any)
- Any sections that need manual review

**Output structure:**
```
{vault}/{project-name}/
├── PROJECT-SPEC.md      # Main spec with Features section
├── ENV.md               # Environment variables (if .env exists)
└── features/            # Feature detail pages (if docs/features/ exists)
    ├── {feature-1}.md
    └── {feature-2}.md
```

</process>

<success_criteria>
This workflow is complete when:
- [ ] Obsidian vault located or path provided
- [ ] Project comprehensively analyzed (8 analysis areas)
- [ ] Project type detected and confirmed by user
- [ ] Core spec generated with all applicable sections
- [ ] Type-specific addendum generated
- [ ] Features section added (if docs/features/ exists)
- [ ] Feature detail pages created in vault/features/
- [ ] Obsidian frontmatter added
- [ ] PROJECT-SPEC.md saved to vault
- [ ] ENV.md saved to vault (if .env exists)
- [ ] User notified of completion and location
</success_criteria>
