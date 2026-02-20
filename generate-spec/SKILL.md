---
name: generate-spec
description: Analyzes codebases and generates specification documents. Auto-detects project type and produces a spec describing what exists. Saves to Notion via API with ENV as a child page. Use when documenting a project or onboarding to a codebase.
---

<essential_principles>

<principle name="audience-first" priority="critical">
**Write for non-technical readers**

The primary audience is Product Management and Product Designers - people who are NOT engineers. Every word must be accessible to someone without a technical background.

- NO technical jargon (no "API endpoints", "middleware", "state management")
- NO code snippets or technical implementation details
- USE plain language: "users see a loading spinner" not "async state renders spinner"
- DESCRIBE what users experience, not how the system works internally
- EXPLAIN features in terms of user value, not technical capability

If you must reference technical concepts, explain them in one simple sentence.
</principle>

<principle name="spec-not-prd">
**A Spec documents what IS, not what we want**

This is NOT a PRD (Product Requirements Document). A spec describes the product as it exists - how it works, what users see, what the flows are. PRDs are for requirements. Specs are for documentation.
</principle>

<principle name="describe-dont-prescribe">
**Describe, don't prescribe**

Write "The user clicks the button and sees a confirmation" NOT "The system shall display a confirmation when the user clicks the button". No FR-XXX IDs. No acceptance criteria. Just clear descriptions.
</principle>

<principle name="flow-based">
**Document flows step-by-step**

Use tables to walk through user flows. Step 1, Step 2, Step 3. This is easy to scan and understand. Include what the user sees at each step.
</principle>

<principle name="notion-output">
**Output to Notion**

All specs are published directly to Notion via the API. The main spec becomes a page under the user-specified parent page, and ENV.md becomes a child page of the spec page.

Notion API Token: Read from `~/.claude/.env` file (look for `NOTION_API_TOKEN=`)
Default parent page (SciPlay): `d841d8f6-efb9-412a-b025-8db02c06ccc8`
Workspace: Invisionnaire LLC
</principle>

</essential_principles>

<configuration>
**Notion Configuration**

| Setting | Value |
|---------|-------|
| API Token | Read from `NOTION_API_TOKEN` env var or `~/.claude/.notion-token` file |
| Workspace | Invisionnaire LLC |
| Default Parent Page | SciPlay (`d841d8f6-efb9-412a-b025-8db02c06ccc8`) |
| API Version | `2022-06-28` |

**Output Structure (Notion pages):**
- `{parent-page} / {Project Name} — Spec` - Main spec page
- `{spec-page} / ENV — Environment Variables` - Child page (if .env exists)
- `{spec-page} / {Feature Name}` - Child pages for each feature (if docs/features/ exists)

**How to publish to Notion:**
Use the Notion API via Python/curl to create pages with block content. The API token is a workspace-level integration ("Clawdbot"). Pages the integration creates are automatically accessible. Use `POST /v1/pages` to create pages and `PATCH /v1/blocks/{id}/children` to append blocks (max 100 blocks per request).

**Feature Specs:**
If the project has `docs/features/FEATURE-*.md` files (created by `feature-spec` skill):
1. Add a **Features** section to the main spec page with summaries
2. Create individual child pages under the spec page for each feature
</configuration>

<intake>
**Ready to generate a project spec.**

The skill will:
1. Analyze the current project comprehensively
2. Auto-detect project type (website, webapp, mobile, game)
3. Generate Core Spec + appropriate addendum
4. Publish to Notion under the specified parent page (default: SciPlay)
5. Create ENV child page if .env exists

**Confirm to proceed, or specify:**
- Different project path
- Override project type
- Different Notion parent page
</intake>

<routing>
| User Response | Action |
|---------------|--------|
| Confirm, "yes", "go", "proceed" | Execute `workflows/generate-spec.md` |
| Specifies path | Set project path, then execute workflow |
| Specifies type | Skip auto-detection, use specified type |
| Specifies output | Override Obsidian path |
| "help", "options" | Show configuration options |

**After confirmation, read and follow `workflows/generate-spec.md` exactly.**
</routing>

<project_types>
| Type | Indicators | Addendum |
|------|-----------|----------|
| **Website** | Static HTML, CMS, no auth, content-focused | templates/website-addendum.md |
| **Webapp** | Auth system, CRUD ops, SaaS features, API | templates/webapp-addendum.md |
| **Mobile** | React Native, Flutter, Swift, Kotlin, .xcodeproj, Android manifests | templates/mobile-addendum.md |
| **Game** | Unity, Unreal, Godot, game loops, sprites, physics | templates/game-addendum.md |

Mixed projects (e.g., webapp with mobile app) get multiple addendums.
</project_types>

<reference_index>
**Domain Knowledge** (in `references/`):

- **project-detection.md** - How to identify project types from codebase
- **analysis-checklist.md** - Comprehensive analysis areas per project type
- **spec-best-practices.md** - Writing effective, verifiable specs
- **llm-optimized-structure.md** - Structure patterns for LLM/AI parsing and human readability
</reference_index>

<templates_index>
**Output Templates** (in `templates/`):

| Template | Purpose |
|----------|---------|
| core-spec.md | Base spec structure (all projects) |
| website-addendum.md | Content, SEO, CMS, IA sections |
| webapp-addendum.md | Auth, workflows, API contracts |
| mobile-addendum.md | Platform, offline, push, app store |
| game-addendum.md | GDD sections (loop, mechanics, economy) |
</templates_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| generate-spec.md | Main workflow: analyze → detect → generate → save |
</workflows_index>

<success_criteria>
Spec generation is complete when:
- [ ] Project thoroughly analyzed (structure, deps, patterns, APIs)
- [ ] Project type correctly identified and confirmed
- [ ] Core Spec generated with all applicable sections
- [ ] Type-specific addendum appended
- [ ] Features section added (if docs/features/ exists)
- [ ] Feature detail child pages created in Notion
- [ ] Main spec page published to Notion under parent page
- [ ] ENV child page published to Notion (if .env exists in project)
- [ ] User informed of Notion page URLs
</success_criteria>
