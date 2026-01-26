---
name: coolify-skill
description: Expert Coolify v4 administrator for managing deployments, diagnosing failures, and scaling services. Use when working with Coolify projects, fixing broken deployments, or deploying applications at scale.
---

<essential_principles>

<authentication>
**Credentials:**

```
COOLIFY_URL: https://coolify.bml.gg
COOLIFY_API_KEY: 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd
```
</authentication>

<api_basics>
**Base URL:** `https://coolify.bml.gg/api/v1`

**Headers:**
```
Authorization: Bearer 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd
Content-Type: application/json
Accept: application/json
```

**Rate Limiting:** Coolify has no documented rate limits, but batch operations sequentially to avoid overwhelming the server.
</api_basics>

<safety_guardrails>
**Before any destructive operation:**
1. Confirm the target resource exists and matches expectations
2. Check current status before changing it
3. For batch operations, list affected resources first and confirm with user
4. Never delete resources without explicit user confirmation
5. Always have a rollback plan for deployments

**Destructive Operations (require confirmation):**
- Deleting applications, services, databases
- Stopping production services
- Force restarting services
- Changing environment variables on running services
- Modifying server configurations
</safety_guardrails>

<resource_hierarchy>
```
Team
└── Projects
    └── Environments (production, staging, etc.)
        └── Resources
            ├── Applications (Git repos, Docker images)
            ├── Databases (PostgreSQL, MySQL, Redis, etc.)
            └── Services (pre-built stacks like WordPress, n8n)
```

Each resource has a UUID for API operations.
</resource_hierarchy>

</essential_principles>

<intake>
What would you like to do?

1. **Diagnose** - Check instance health, list projects, identify failing deployments
2. **Fix** - Troubleshoot and fix broken deployments
3. **Deploy** - Deploy or redeploy applications and services
4. **Manage** - Start, stop, restart services; manage environment variables

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow | Description |
|----------|----------|-------------|
| 1, "diagnose", "health", "list", "status", "check" | workflows/diagnose.md | Health checks, list resources, find failures |
| 2, "fix", "broken", "failing", "error", "troubleshoot" | workflows/fix-deployment.md | Diagnose and fix deployment failures |
| 3, "deploy", "redeploy", "ship", "release" | workflows/deploy.md | Deploy or redeploy applications |
| 4, "manage", "start", "stop", "restart", "env" | workflows/manage.md | Service lifecycle and configuration |

**After reading the workflow, follow it exactly.**
</routing>

<quick_reference>
**Common API Calls:**

```bash
# Check API connection
curl -s "https://coolify.bml.gg/api/v1/version" -H "Authorization: Bearer 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd"

# List all projects
curl -s "https://coolify.bml.gg/api/v1/projects" -H "Authorization: Bearer 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd"

# List all applications
curl -s "https://coolify.bml.gg/api/v1/applications" -H "Authorization: Bearer 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd"

# Deploy application by UUID
curl -X POST "https://coolify.bml.gg/api/v1/applications/{uuid}/deploy" \
  -H "Authorization: Bearer 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd"

# Restart application
curl -X POST "https://coolify.bml.gg/api/v1/applications/{uuid}/restart" \
  -H "Authorization: Bearer 6|0oBX1hVqhHBdZ0tnPSlTgBAKjoRGOqiFLfET5hfu5a8b7edd"
```
</quick_reference>

<reference_index>
All domain knowledge in `references/`:

**API:** api-endpoints.md - Complete endpoint reference
**Troubleshooting:** troubleshooting.md - Common failures and fixes
**Deployment:** deployment-patterns.md - Best practices for deployments
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| diagnose.md | Health checks, list resources, identify issues |
| fix-deployment.md | Troubleshoot and repair broken deployments |
| deploy.md | Deploy applications and services |
| manage.md | Service lifecycle and configuration management |
</workflows_index>

<success_criteria>
Operations are successful when:
- API calls return expected status codes (200/201 for success)
- Deployments show status "running" or "healthy"
- No error messages in deployment logs
- User confirms the expected outcome
</success_criteria>
