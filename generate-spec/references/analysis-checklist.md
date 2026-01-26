# Comprehensive Analysis Checklist

<overview>
This checklist ensures thorough project analysis before spec generation. Each area contributes to specific spec sections.
</overview>

<analysis_areas>

## 1. Project Structure

**What to find:**
- Root directory organization
- Source code location (`/src`, `/app`, `/lib`)
- Config files location
- Build output directories
- Documentation location

**How to analyze:**
```bash
# Directory tree (depth 3)
find . -type d -maxdepth 3 | head -50

# Key files in root
ls -la

# Source structure
find ./src -type f -name "*.{js,ts,py,php,rb}" | head -30
```

**Contributes to:** Scope, Architecture sections

---

## 2. Technology Stack

**What to find:**
- Primary language(s)
- Framework(s) and version(s)
- Key dependencies
- Dev dependencies
- Build tools

**How to analyze:**
```bash
# JavaScript/Node
cat package.json | jq '.dependencies, .devDependencies'

# Python
cat requirements.txt || cat Pipfile || cat pyproject.toml

# PHP
cat composer.json | jq '.require, .require-dev'

# Ruby
cat Gemfile

# Rust
cat Cargo.toml

# Go
cat go.mod
```

**Contributes to:** Executive Summary, Technical Constraints

---

## 3. Architecture Patterns

**What to find:**
- Overall architecture (MVC, MVVM, Clean, etc.)
- Component organization
- State management approach
- API design patterns

**Patterns to look for:**
- `/controllers/`, `/models/`, `/views/` → MVC
- `/components/`, `/hooks/`, `/context/` → React patterns
- `/domain/`, `/application/`, `/infrastructure/` → Clean Architecture
- `/features/`, `/modules/` → Feature-based
- `/entities/`, `/systems/`, `/components/` → ECS (games)

**Contributes to:** Architecture, UX/UI sections

---

## 4. Data Layer

**What to find:**
- Database type and name
- ORM/query builder
- Schema/migrations
- Seed data
- Data relationships

**How to analyze:**
```bash
# Look for migrations
find . -path "*migration*" -name "*.{js,ts,php,py,rb,sql}"

# Look for models/entities
find . -path "*model*" -o -path "*entity*" | head -20

# Check for schema files
find . -name "schema.*" -o -name "*.prisma" -o -name "*.graphql"

# Environment for DB connection
grep -i "database\|mysql\|postgres\|mongo" .env* 2>/dev/null
```

**Contributes to:** Data & Integrations section

---

## 5. Authentication & Authorization

**What to find:**
- Auth mechanism (session, JWT, OAuth)
- User model/table
- Roles and permissions
- Protected routes/middleware
- Auth providers (Auth0, Clerk, etc.)

**How to analyze:**
```bash
# Auth-related files
find . -iname "*auth*" -o -iname "*login*" -o -iname "*session*"

# Middleware
find . -path "*middleware*" -name "*.{js,ts,php}"

# Permission/role files
find . -iname "*permission*" -o -iname "*role*" -o -iname "*policy*"
```

**Contributes to:** Requirements (FR), Webapp Addendum

---

## 6. External Integrations

**What to find:**
- Third-party APIs used
- Payment processors
- Email services
- Analytics/tracking
- Cloud services

**How to analyze:**
```bash
# Check environment variables
grep -E "API_KEY|SECRET|TOKEN" .env* 2>/dev/null | sed 's/=.*/=***/'

# Check for SDK imports
grep -r "stripe\|twilio\|sendgrid\|mailgun\|aws\|google\|firebase" --include="*.{js,ts,php,py}" | head -20

# Check package dependencies
grep -E "stripe|twilio|aws-sdk|firebase|@google" package.json composer.json 2>/dev/null
```

**Contributes to:** Data & Integrations, Requirements

---

## 7. Testing Setup

**What to find:**
- Test framework(s)
- Test file locations
- Test coverage config
- CI/CD configuration

**How to analyze:**
```bash
# Test directories
find . -type d -name "*test*" -o -name "*spec*" | head -10

# Test config files
ls jest.config.* vitest.config.* phpunit.xml pytest.ini 2>/dev/null

# CI/CD
ls .github/workflows/* .gitlab-ci.yml .circleci/config.yml 2>/dev/null

# Test scripts in package.json
grep -A5 '"scripts"' package.json | grep -i test
```

**Contributes to:** Testing & Acceptance section

---

## 8. Key Features

**What to find:**
- Main user-facing features
- Core business logic
- Feature flags
- Notable implementations

**How to analyze:**
```bash
# Routes/endpoints (framework-specific)
# Laravel
grep -r "Route::" routes/ 2>/dev/null | head -20

# Express/Node
grep -rE "app\.(get|post|put|delete|patch)" --include="*.{js,ts}" | head -20

# Check README for feature list
head -100 README.md

# Look for feature flags
grep -ri "feature\|flag\|toggle" --include="*.{js,ts,php}" | head -10
```

**Contributes to:** Scope, Requirements (FR)

</analysis_areas>

<type_specific_analysis>

## Website-Specific Analysis

Additional checks:
- [ ] Sitemap structure
- [ ] SEO setup (meta, schema, robots.txt)
- [ ] CMS configuration
- [ ] Content types/templates
- [ ] Image optimization setup
- [ ] Performance config (caching, CDN)

## Webapp-Specific Analysis

Additional checks:
- [ ] User roles/permissions matrix
- [ ] API endpoint inventory
- [ ] Background job setup
- [ ] Caching strategy
- [ ] Session management
- [ ] Rate limiting

## Mobile-Specific Analysis

Additional checks:
- [ ] Supported OS versions
- [ ] Required permissions (manifest)
- [ ] Offline capabilities
- [ ] Push notification setup
- [ ] Deep linking config
- [ ] App store metadata

## Game-Specific Analysis

Additional checks:
- [ ] Game loop structure
- [ ] Input handling
- [ ] Scene/level organization
- [ ] Asset pipeline
- [ ] Save system
- [ ] Multiplayer/networking

</type_specific_analysis>

<output_mapping>

## Analysis to Spec Section Mapping

| Analysis Area | Core Spec Section |
|---------------|-------------------|
| Project Structure | Scope, Architecture |
| Technology Stack | Executive Summary, Constraints |
| Architecture | UX/UI, Technical Design |
| Data Layer | Data & Integrations |
| Auth/Authz | Requirements (FR), Security NFRs |
| Integrations | Data & Integrations |
| Testing | Testing & Acceptance |
| Key Features | Scope, Requirements (FR) |

</output_mapping>
