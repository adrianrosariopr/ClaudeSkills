---
name: backend-skill
description: Expert backend developer for building robust server-side applications. Build APIs, manage databases, implement authentication, and optimize performance across PHP/Laravel, Node.js/Express, Python/Django/FastAPI, and Go. Full lifecycle coverage from architecture through deployment.
---

<essential_principles>

<principle name="api-first-design">
Design APIs before implementation. Clear contracts enable parallel frontend/backend development and better testing:
- Define endpoints, request/response schemas, and error formats first
- Use OpenAPI/Swagger for documentation
- Version APIs from day one (`/api/v1/`)
- Consistent response structure across all endpoints
</principle>

<principle name="security-is-foundational">
Security cannot be bolted on later. Every endpoint must consider:
- Input validation and sanitization (never trust client data)
- Authentication and authorization on every protected route
- Rate limiting to prevent abuse
- SQL injection, XSS, CSRF protection
- Secrets in environment variables, never in code
</principle>

<principle name="database-is-the-bottleneck">
Most performance issues trace back to the database:
- Use indexes strategically (but not on everything)
- Avoid N+1 queries (eager load relationships)
- Write queries that explain well (`EXPLAIN ANALYZE`)
- Cache aggressively at the right layer
- Migrations are code - version control them
</principle>

<principle name="fail-gracefully">
Production systems will fail. Design for it:
- Comprehensive error handling with meaningful messages
- Structured logging (JSON) for debugging
- Health checks and monitoring endpoints
- Graceful degradation when dependencies fail
- Circuit breakers for external services
</principle>

<principle name="stateless-by-default">
Stateless services scale horizontally:
- Store session data in external stores (Redis, database)
- Use JWTs or tokens instead of server-side sessions
- Avoid in-memory state that can't be shared
- Design for multiple instances behind a load balancer
</principle>

</essential_principles>

<intake>
## What framework/language are you using?

1. **PHP / Laravel** - Eloquent ORM, Spatie permissions, queues, Sanctum
2. **Node.js / Express / Fastify** - REST/GraphQL APIs, Prisma, TypeScript
3. **Python / Django / FastAPI** - Django ORM, async Python, Pydantic
4. **Go / Gin / Echo** - High-performance APIs, Go patterns
5. **Framework-agnostic** - General backend patterns, architecture

**After framework selection, what would you like help with?**

1. **API Expert** - REST design, GraphQL, versioning, documentation
2. **Database Expert** - Schema design, migrations, query optimization, indexing
3. **Auth Expert** - Authentication, authorization, OAuth, sessions, tokens
4. **Build Endpoint** - Create a new API endpoint from scratch
5. **Debug Backend** - Fix server errors, performance issues, bugs
6. **Optimize Performance** - Caching, queues, profiling, scaling
7. **Something else** - Describe what you need

**Wait for response before proceeding.**
</intake>

<routing>
## Framework Selection

| Response | Framework | References to Load |
|----------|-----------|-------------------|
| 1, "php", "laravel" | PHP/Laravel | `references/laravel/*.md` (all Laravel refs) |
| 2, "node", "nodejs", "express", "fastify", "typescript" | Node.js | `references/nodejs/patterns.md` |
| 3, "python", "django", "fastapi", "flask" | Python | `references/python/patterns.md` |
| 4, "go", "golang", "gin", "echo" | Go | `references/go/patterns.md` |
| 5, "general", "agnostic", "architecture" | Core only | Core references only |

## Task Routing

| Response | Workflow |
|----------|----------|
| 1, "api", "rest", "graphql", "endpoint design", "versioning" | `workflows/api-expert.md` |
| 2, "database", "schema", "migration", "query", "index", "sql" | `workflows/database-expert.md` |
| 3, "auth", "authentication", "authorization", "oauth", "jwt", "session" | `workflows/auth-expert.md` |
| 4, "build", "create", "new endpoint", "implement" | `workflows/build-endpoint.md` |
| 5, "debug", "fix", "error", "500", "broken", "crash" | `workflows/debug-backend.md` |
| 6, "slow", "optimize", "performance", "cache", "queue", "scale" | `workflows/optimize-performance.md` |
| 7, other | Clarify intent, then route to appropriate workflow |

**After identifying framework and task:**
1. Read the framework-specific references first
2. Read the workflow file
3. Always include relevant core references (api-design, security, etc.)

</routing>

<verification_loop>
After every change, verify:

**PHP/Laravel:**
```bash
# 1. Does it pass static analysis?
./vendor/bin/phpstan analyse

# 2. Do tests pass?
php artisan test

# 3. Check for issues
php artisan route:list
php artisan config:clear && php artisan cache:clear
```

**Node.js:**
```bash
# 1. Does it compile/lint?
npm run lint && npm run build

# 2. Do tests pass?
npm test

# 3. Check types
npm run typecheck  # or tsc --noEmit
```

**Python:**
```bash
# 1. Does it pass linting?
ruff check . && mypy .

# 2. Do tests pass?
pytest

# 3. Check migrations
python manage.py check  # Django
```

**Go:**
```bash
# 1. Does it compile?
go build ./...

# 2. Do tests pass?
go test ./...

# 3. Check for issues
go vet ./...
```

Report to the user:
- "Build/Lint: OK"
- "Tests: X pass, Y fail"
- "Ready for you to test [specific endpoint]"
</verification_loop>

<reference_index>
## Domain Knowledge

### Core (Framework-Agnostic)
All in `references/core/`:
- api-design.md - REST principles, GraphQL, versioning, error handling
- database-patterns.md - Schema design, indexing, migrations, ORMs
- authentication.md - Auth patterns, sessions, JWTs, OAuth 2.0
- caching.md - Redis, application caching, cache invalidation
- queues.md - Background jobs, message queues, async processing
- security.md - OWASP top 10, input validation, rate limiting
- testing.md - Unit, integration, E2E testing patterns
- performance.md - Profiling, optimization, scaling strategies
- anti-patterns.md - Common backend mistakes and how to avoid them

### PHP / Laravel
All in `references/laravel/`:
- patterns.md - Laravel 12, Eloquent, controllers, services, queues, events
- spatie-permissions.md - Roles, permissions, policies, gates, middleware
- sanctum-auth.md - API authentication, tokens, guards, SPA auth
- multi-tenancy.md - Team-based permissions, scoped roles
- testing.md - Testing authorization with PHPUnit/Pest
- permission-caching.md - Cache behavior, common issues, deployment
- deployment.md - Migrations vs seeders, deploy scripts, rollbacks
- performance.md - N+1 prevention, eager loading, query optimization

### Node.js
All in `references/nodejs/`:
- patterns.md - Express/Fastify, Prisma, TypeScript, async patterns

### Python
All in `references/python/`:
- patterns.md - Django, FastAPI, SQLAlchemy, async Python

### Go
All in `references/go/`:
- patterns.md - Gin/Echo, GORM, Go idioms, concurrency
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| api-expert.md | Advanced API design and implementation |
| database-expert.md | Schema design, queries, optimization |
| auth-expert.md | Authentication and authorization patterns |
| build-endpoint.md | Create new endpoints from scratch |
| debug-backend.md | Fix server errors and issues |
| optimize-performance.md | Caching, queues, profiling |
</workflows_index>

<quick_reference>
## HTTP Status Codes

| Code | When to Use |
|------|-------------|
| 200 | Success (GET, PUT, PATCH) |
| 201 | Created (POST) |
| 204 | No Content (DELETE) |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (not authenticated) |
| 403 | Forbidden (authenticated but not allowed) |
| 404 | Not Found |
| 409 | Conflict (duplicate resource) |
| 422 | Unprocessable Entity (semantic error) |
| 429 | Too Many Requests (rate limited) |
| 500 | Internal Server Error |

## Standard Response Format

```json
{
  "data": { ... },
  "meta": { "page": 1, "total": 100 },
  "errors": null
}

{
  "data": null,
  "errors": [
    { "code": "VALIDATION_ERROR", "message": "Email is required", "field": "email" }
  ]
}
```
</quick_reference>

<context7_usage>
## Getting Current Documentation

**Laravel:**
```
mcp__context7__query-docs:
  libraryId: "/laravel/docs"
  query: "your question"
```

**Express:**
```
mcp__context7__query-docs:
  libraryId: "/expressjs/express"
  query: "your question"
```

**Django:**
```
mcp__context7__query-docs:
  libraryId: "/django/django"
  query: "your question"
```

**FastAPI:**
```
mcp__context7__query-docs:
  libraryId: "/tiangolo/fastapi"
  query: "your question"
```
</context7_usage>
