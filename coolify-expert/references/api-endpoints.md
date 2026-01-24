<overview>
Complete reference for Coolify v4 API endpoints. All endpoints require Bearer token authentication.

**Base URL:** `${COOLIFY_URL}/api/v1`
**Authentication:** `Authorization: Bearer ${COOLIFY_API_KEY}`
</overview>

<version>
<endpoint method="GET" path="/version">
Returns Coolify version information.

**Response:**
```json
{
  "version": "4.0.0-beta.462"
}
```

**Use case:** Verify API connectivity and version compatibility.
</endpoint>
</version>

<projects>
<endpoint method="GET" path="/projects">
List all projects accessible by the API token.

**Response:**
```json
[
  {
    "id": 1,
    "uuid": "project-uuid-here",
    "name": "My Project",
    "description": "Project description"
  }
]
```
</endpoint>

<endpoint method="GET" path="/projects/{uuid}">
Get details for a specific project.
</endpoint>

<endpoint method="POST" path="/projects">
Create a new project.

**Body:**
```json
{
  "name": "Project Name",
  "description": "Optional description"
}
```
</endpoint>

<endpoint method="DELETE" path="/projects/{uuid}">
Delete a project. **Destructive** - removes all resources in the project.
</endpoint>
</projects>

<applications>
<endpoint method="GET" path="/applications">
List all applications.

**Response fields:**
- `uuid` - Unique identifier for API operations
- `name` - Application name
- `status` - Current status (running, exited, failed, building)
- `fqdn` - Fully qualified domain name (URL)
- `git_repository` - Source repository (for git-based apps)
- `git_branch` - Branch being deployed
- `build_pack` - Build method (nixpacks, dockerfile, compose)
- `project_uuid` - Parent project
</endpoint>

<endpoint method="GET" path="/applications/{uuid}">
Get detailed application information including environment variables, resource limits, and configuration.
</endpoint>

<endpoint method="POST" path="/applications">
Create a new application.

**Body (Docker Image):**
```json
{
  "project_uuid": "project-uuid",
  "server_uuid": "server-uuid",
  "environment_name": "production",
  "docker_registry_image_name": "nginx",
  "docker_registry_image_tag": "latest",
  "instant_deploy": true
}
```

**Body (Git Repository):**
```json
{
  "project_uuid": "project-uuid",
  "server_uuid": "server-uuid",
  "environment_name": "production",
  "git_repository": "https://github.com/user/repo",
  "git_branch": "main",
  "build_pack": "nixpacks",
  "instant_deploy": true
}
```
</endpoint>

<endpoint method="PATCH" path="/applications/{uuid}">
Update application configuration.

**Common updatable fields:**
```json
{
  "name": "new-name",
  "fqdn": "https://app.example.com",
  "git_branch": "develop",
  "docker_registry_image_tag": "v2.0",
  "limits_memory": "512M",
  "limits_cpus": "1",
  "health_check_enabled": true,
  "health_check_path": "/health",
  "environment_variables_preview": [
    {"key": "VAR", "value": "value", "is_build_time": false}
  ]
}
```
</endpoint>

<endpoint method="DELETE" path="/applications/{uuid}">
Delete an application. **Destructive** - removes container and configuration.
</endpoint>

<endpoint method="POST" path="/applications/{uuid}/deploy">
Trigger a deployment.

**Query params:**
- `force=true` - Ignore build cache

**Response:**
```json
{
  "deployment_uuid": "deployment-uuid"
}
```
</endpoint>

<endpoint method="POST" path="/applications/{uuid}/start">
Start a stopped application.
</endpoint>

<endpoint method="POST" path="/applications/{uuid}/stop">
Stop a running application.
</endpoint>

<endpoint method="POST" path="/applications/{uuid}/restart">
Restart application without full redeploy.
</endpoint>

<endpoint method="GET" path="/applications/{uuid}/logs">
Get application container logs.

**Query params:**
- `since` - Time filter (e.g., "1h", "30m")
- `tail` - Number of lines
</endpoint>
</applications>

<services>
<endpoint method="GET" path="/services">
List all services (pre-built stacks like WordPress, n8n, etc.).
</endpoint>

<endpoint method="GET" path="/services/{uuid}">
Get service details.
</endpoint>

<endpoint method="POST" path="/services/{uuid}/start">
Start a service.
</endpoint>

<endpoint method="POST" path="/services/{uuid}/stop">
Stop a service.
</endpoint>

<endpoint method="POST" path="/services/{uuid}/restart">
Restart a service.
</endpoint>
</services>

<databases>
<endpoint method="GET" path="/databases">
List all databases.

**Supported types:** PostgreSQL, MySQL, MariaDB, MongoDB, Redis, etc.
</endpoint>

<endpoint method="GET" path="/databases/{uuid}">
Get database details including connection strings.
</endpoint>

<endpoint method="POST" path="/databases/{uuid}/start">
Start a database.
</endpoint>

<endpoint method="POST" path="/databases/{uuid}/stop">
Stop a database.
</endpoint>

<endpoint method="POST" path="/databases/{uuid}/restart">
Restart a database.
</endpoint>
</databases>

<servers>
<endpoint method="GET" path="/servers">
List all servers.

**Response fields:**
- `uuid` - Server identifier
- `name` - Server name
- `ip` - Server IP address
- `is_reachable` - Whether Coolify can connect
- `is_usable` - Whether server is properly configured
</endpoint>

<endpoint method="GET" path="/servers/{uuid}">
Get server details.
</endpoint>

<endpoint method="GET" path="/servers/{uuid}/resources">
Get all resources (apps, services, databases) on a server.
</endpoint>
</servers>

<teams>
<endpoint method="GET" path="/teams">
List teams accessible by the API token.
</endpoint>

<endpoint method="GET" path="/teams/{id}">
Get team details.
</endpoint>

<endpoint method="GET" path="/teams/{id}/members">
List team members.
</endpoint>
</teams>

<http_status_codes>
| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad request (invalid params) |
| 401 | Unauthorized (invalid token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Resource not found |
| 422 | Validation error |
| 500 | Server error |
</http_status_codes>
