# Claude Code Skills

Custom skills for [Claude Code](https://claude.ai/code) that extend its capabilities with domain-specific expertise.

## Quick Setup (New Machine)

```bash
# 1. Clone the repo
git clone git@github.com:<your-username>/ClaudeSkills.git

# 2. Copy skills to the global Claude folder
rsync -av --exclude='.git' ./ClaudeSkills/ ~/.claude/skills/

# 3. Create your secrets file from the template
cp .env.example ~/.claude/.env

# 4. Edit ~/.claude/.env and fill in your API keys
nano ~/.claude/.env
```

That's it. Open Claude Code and your skills are available.

## How It Works

### Skill Loading

Claude Code automatically loads skills from `~/.claude/skills/`. Each skill is a folder with:

```
skill-name/
├── SKILL.md              # Skill definition (name, description, instructions)
├── references/           # Domain knowledge files the skill reads
│   └── *.md
├── workflows/            # Step-by-step procedures
│   └── *.md
└── templates/            # Output templates (optional)
    └── *.md
```

When you invoke a skill (e.g., `/generate-spec`), Claude loads `SKILL.md` first, then reads references and workflows as needed.

### Secrets Management

API keys are stored in `~/.claude/.env` — a single file on your machine that is **never committed** to any repo.

```
# ~/.claude/.env
NOTION_API_TOKEN=ntn_xxxxx
MAILGUN_API_KEY=key-xxxxx
COOLIFY_API_TOKEN=xxxxx
```

Skills that need API keys are configured to read from this file. When Claude runs a skill, it reads `~/.claude/.env` to get the required keys.

**Why this approach?**
- Keys live outside any repo — no risk of accidental commits
- One file for all skills — easy to set up on a new machine
- GitHub push protection won't block you — secrets never touch git

### Updating Skills

When you modify skills locally (in `~/.claude/skills/`), sync them back to this repo:

```bash
# From skills → repo (after editing skills locally)
rsync -av --exclude='.git' ~/.claude/skills/ /path/to/ClaudeSkills/

# From repo → skills (after pulling updates)
rsync -av --exclude='.git' /path/to/ClaudeSkills/ ~/.claude/skills/
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `backend-skill` | Server-side development (Laravel, Node.js, Python, Go) |
| `frontend-skill` | UI/UX development (React, Vue, Tailwind, React Native) |
| `database-skill` | Database architecture and optimization |
| `stripe-skill` | Stripe payments integration and management |
| `mailgun-skill` | Mailgun email delivery integration |
| `notion-skill` | Notion page and database management via MCP |
| `cloudflare-skill` | Cloudflare DNS, security, Workers, and Zero Trust |
| `coolify-skill` | Coolify v4 deployment management |
| `seo-skill` | SEO and web accessibility optimization |
| `gsap-skill` | GSAP animations and ScrollTrigger |
| `simplify-code` | Code simplification and cleanup |
| `generate-spec` | Generate project specs and publish to Notion |
| `generate-feature` | Generate feature implementation plans |

## Required Secrets by Skill

| Skill | Key | Where to Get It |
|-------|-----|-----------------|
| `generate-spec` | `NOTION_API_TOKEN` | [Notion Integrations](https://www.notion.so/my-integrations) |
| `notion-skill` | Uses Notion MCP server | Configure in `~/.claude/settings.local.json` |

Most skills don't need API keys — they provide expertise and patterns, not API access.
