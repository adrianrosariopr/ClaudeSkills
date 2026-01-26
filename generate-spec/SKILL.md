---
name: generate-spec
description: Analyzes codebases and generates specification documents. Auto-detects project type and produces a spec describing what exists. Saves to Obsidian vault with ENV.md. Use when documenting a project or onboarding to a codebase.
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

<principle name="obsidian-output">
**Output to Obsidian vault**

All specs save to the configured Obsidian vault with proper frontmatter for Dataview queries, tags, and wikilinks.

Vault path: `/Users/adrian-rosario/Library/CloudStorage/GoogleDrive-adrianrosariopr@gmail.com/My Drive/Obsidian`
</principle>

</essential_principles>

<configuration>
**Obsidian Vault Paths**

| Platform | Path |
|----------|------|
| Mac | `/Users/adrian-rosario/Library/CloudStorage/GoogleDrive-adrianrosariopr@gmail.com/My Drive/Obsidian` |
| Windows | `TODO` |

**Output Files:**
- `{vault}/{project-name}/PROJECT-SPEC.md` - Project specification
- `{vault}/{project-name}/ENV.md` - Environment variables (if .env exists)
- `{vault}/{project-name}/features/*.md` - Feature detail pages (if docs/features/ exists)

**Feature Specs:**
If the project has `docs/features/FEATURE-*.md` files (created by `feature-spec` skill):
1. Add a **Features** section to PROJECT-SPEC.md with summaries
2. Create individual pages in `{vault}/{project-name}/features/` with full details
3. Link summaries to detail pages using wikilinks
</configuration>

<intake>
**Ready to generate a project spec.**

The skill will:
1. Analyze the current project comprehensively
2. Auto-detect project type (website, webapp, mobile, game)
3. Generate Core Spec + appropriate addendum
4. Save to your Obsidian vault

**Confirm to proceed, or specify:**
- Different project path
- Override project type
- Custom output location
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
- [ ] Feature detail pages created in vault
- [ ] Obsidian frontmatter added (tags, dates, links)
- [ ] PROJECT-SPEC.md saved to Obsidian vault
- [ ] ENV.md saved to vault (if .env exists in project)
- [ ] User informed of output location
</success_criteria>
