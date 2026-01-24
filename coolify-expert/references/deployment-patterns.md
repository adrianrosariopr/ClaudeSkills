<overview>
Best practices and patterns for deploying applications at scale with Coolify.
</overview>

<deployment_strategies>

<strategy name="rolling_deployment">
**When to use:** Production apps that need zero downtime

Coolify handles rolling deployments automatically when health checks are configured. New container starts, passes health check, then old container is removed.

**Configuration:**
```bash
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "health_check_enabled": true,
    "health_check_path": "/health",
    "health_check_interval": 10,
    "health_check_timeout": 5,
    "health_check_retries": 3,
    "health_check_start_period": 30
  }'
```
</strategy>

<strategy name="blue_green">
**When to use:** When you need instant rollback capability

Coolify doesn't have native blue-green, but you can simulate:
1. Create two identical applications (blue and green)
2. Point domain to active one
3. Deploy to inactive, verify
4. Switch domain to newly deployed

**Implementation:**
```bash
# Deploy to green (inactive)
curl -X POST "${COOLIFY_URL}/api/v1/applications/{green-uuid}/deploy" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# After verification, switch domain
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{green-uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"fqdn": "https://app.example.com"}'

# Remove domain from blue
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{blue-uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"fqdn": ""}'
```
</strategy>

<strategy name="canary">
**When to use:** Gradually rolling out risky changes

Deploy new version to subset of traffic using weighted routing (requires external load balancer or Coolify's Traefik config).
</strategy>

</deployment_strategies>

<batch_deployment_patterns>

<pattern name="sequential_safe">
**Best for:** Production environments

Deploy one at a time with verification between each:

```bash
UUIDS=("uuid1" "uuid2" "uuid3")

for uuid in "${UUIDS[@]}"; do
  echo "Deploying: $uuid"

  # Deploy
  curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}"

  # Wait for deployment to complete
  while true; do
    status=$(curl -s "${COOLIFY_URL}/api/v1/applications/${uuid}" \
      -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq -r '.status')

    if [ "$status" = "running" ]; then
      echo "$uuid is running"
      break
    elif [ "$status" = "failed" ]; then
      echo "$uuid FAILED - stopping batch"
      exit 1
    fi

    sleep 10
  done
done
```
</pattern>

<pattern name="parallel_fast">
**Best for:** Non-critical environments, staging

Deploy all at once for speed:

```bash
UUIDS=("uuid1" "uuid2" "uuid3")

# Trigger all deployments
for uuid in "${UUIDS[@]}"; do
  curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}" &
done
wait

# Then verify all
for uuid in "${UUIDS[@]}"; do
  status=$(curl -s "${COOLIFY_URL}/api/v1/applications/${uuid}" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq -r '.status')
  echo "$uuid: $status"
done
```
</pattern>

<pattern name="dependency_ordered">
**Best for:** Apps with dependencies (database → backend → frontend)

Deploy in dependency order:

```bash
# 1. Deploy databases first
DB_UUIDS=("db-uuid1" "db-uuid2")
for uuid in "${DB_UUIDS[@]}"; do
  curl -X POST "${COOLIFY_URL}/api/v1/databases/${uuid}/restart" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}"
done
sleep 30  # Wait for DBs

# 2. Deploy backend services
BACKEND_UUIDS=("api-uuid" "worker-uuid")
for uuid in "${BACKEND_UUIDS[@]}"; do
  curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}"
done
sleep 60  # Wait for backends

# 3. Deploy frontends
FRONTEND_UUIDS=("web-uuid" "admin-uuid")
for uuid in "${FRONTEND_UUIDS[@]}"; do
  curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}"
done
```
</pattern>

</batch_deployment_patterns>

<environment_management>

<pattern name="env_sync">
**Sync env vars from staging to production:**

```bash
STAGING_UUID="staging-uuid"
PROD_UUID="prod-uuid"

# Get staging env vars
ENVS=$(curl -s "${COOLIFY_URL}/api/v1/applications/${STAGING_UUID}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.environment_variables')

# Apply to production (review first!)
echo "$ENVS" | jq '.'

# After review, apply
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/${PROD_UUID}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{\"environment_variables_preview\": $ENVS}"
```
</pattern>

<pattern name="secret_rotation">
**Rotate a secret across all apps:**

```bash
NEW_SECRET="new-rotated-secret-value"
KEY_NAME="DATABASE_PASSWORD"

# Find all apps using this secret
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq --arg key "$KEY_NAME" '.[] | select(.environment_variables[]?.key == $key) | .uuid' -r | \
  while read uuid; do
    echo "Updating $uuid"
    # Update env var and restart
    # (Implementation depends on how Coolify handles env var updates)
  done
```
</pattern>

</environment_management>

<monitoring_patterns>

<pattern name="health_dashboard">
**Quick health check script:**

```bash
echo "=== Coolify Health Check ==="
echo "Timestamp: $(date)"
echo ""

# Version
echo "Version:"
curl -s "${COOLIFY_URL}/api/v1/version" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq -r '.version'
echo ""

# Servers
echo "Servers:"
curl -s "${COOLIFY_URL}/api/v1/servers" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq -r '.[] | "\(.name): reachable=\(.is_reachable), usable=\(.is_usable)"'
echo ""

# Applications
echo "Applications:"
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq -r 'group_by(.status) | .[] | "\(.[0].status): \(length)"'
echo ""

# Failing apps
echo "Failing Applications:"
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq -r '.[] | select(.status != "running" and .status != null) | "  - \(.name) (\(.status))"'
```
</pattern>

</monitoring_patterns>

<rollback_patterns>

<pattern name="git_rollback">
**Rollback git-based app to previous commit:**

```bash
UUID="app-uuid"
PREVIOUS_COMMIT="abc123def456"  # Get from git history

# Update to previous commit
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/${UUID}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{\"git_commit_sha\": \"${PREVIOUS_COMMIT}\"}"

# Redeploy
curl -X POST "${COOLIFY_URL}/api/v1/applications/${UUID}/deploy" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```
</pattern>

<pattern name="image_rollback">
**Rollback Docker image app to previous tag:**

```bash
UUID="app-uuid"
PREVIOUS_TAG="v1.2.3"

curl -X PATCH "${COOLIFY_URL}/api/v1/applications/${UUID}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{\"docker_registry_image_tag\": \"${PREVIOUS_TAG}\"}"

curl -X POST "${COOLIFY_URL}/api/v1/applications/${UUID}/deploy" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```
</pattern>

</rollback_patterns>

<scaling_considerations>

<consideration name="resource_limits">
Always set resource limits to prevent runaway containers:

```bash
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "limits_memory": "512M",
    "limits_cpus": "1",
    "limits_memory_swap": "1G"
  }'
```
</consideration>

<consideration name="staggered_deploys">
For large fleets, stagger deployments to avoid overwhelming servers:
- Deploy 10-20% at a time
- Wait for health checks to pass
- Monitor server resources between batches
</consideration>

<consideration name="backup_before_major">
Before major deployments:
1. Ensure database backups are current
2. Document current working state
3. Have rollback commands ready
4. Notify stakeholders
</consideration>

</scaling_considerations>
