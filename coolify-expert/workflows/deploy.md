<required_reading>
**Read these reference files as needed:**
- references/api-endpoints.md - For deployment endpoints
- references/deployment-patterns.md - Best practices
</required_reading>

<process>

<step name="identify_target">
**Step 1: Identify Deployment Target**

Determine what to deploy:
- Single application by name or UUID
- Multiple applications (batch deployment)
- All applications in a project
- Service (pre-built stack)

```bash
# List all applications to find target
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '.[] | {name, uuid, status, fqdn}'
```

**Get the UUID(s) of target application(s).**
</step>

<step name="pre_deploy_check">
**Step 2: Pre-Deployment Checks**

Before deploying, verify:

```bash
# Check current status
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '{name, status, git_branch, git_commit_sha}'
```

**Checklist:**
- Current status is stable (not mid-deployment)
- Git branch/commit looks correct (for git-based apps)
- No pending deployments in queue
</step>

<step name="deploy_single">
**Step 3a: Deploy Single Application**

```bash
# Standard deploy
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/deploy" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Force deploy (ignore build cache)
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/deploy?force=true" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**Response contains deployment UUID for tracking.**
</step>

<step name="deploy_batch">
**Step 3b: Batch Deploy Multiple Applications**

For deploying multiple applications sequentially:

```bash
# Define UUIDs to deploy
UUIDS=("uuid1" "uuid2" "uuid3")

for uuid in "${UUIDS[@]}"; do
  echo "Deploying: $uuid"
  curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}"

  # Wait between deployments to avoid overwhelming server
  sleep 5
done
```

**For parallel deployment (use with caution on resource-constrained servers):**

```bash
for uuid in "${UUIDS[@]}"; do
  curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
    -H "Authorization: Bearer ${COOLIFY_API_KEY}" &
done
wait
```
</step>

<step name="deploy_project">
**Step 3c: Deploy All Applications in Project**

```bash
# Get all applications in project
PROJECT_UUID="your-project-uuid"

# Get project applications (need to filter by project)
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq --arg proj "$PROJECT_UUID" '.[] | select(.project_uuid == $proj) | .uuid' -r | \
  while read uuid; do
    echo "Deploying: $uuid"
    curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/deploy" \
      -H "Authorization: Bearer ${COOLIFY_API_KEY}"
    sleep 5
  done
```
</step>

<step name="monitor_deployment">
**Step 4: Monitor Deployment Progress**

```bash
# Check deployment status
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '{name, status, updated_at}'

# Watch deployment progress (run repeatedly)
watch -n 5 'curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq .status'
```

**Deployment states:**
- `building` - Build in progress
- `deploying` - Container starting
- `running` - Successfully deployed
- `failed` - Deployment failed (check logs)
</step>

<step name="post_deploy_verify">
**Step 5: Post-Deployment Verification**

```bash
# Verify application is running
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '{status, fqdn}'

# Check application responds (if FQDN available)
FQDN=$(curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq -r '.fqdn')

curl -s -o /dev/null -w "%{http_code}" "https://${FQDN}"
```

**Expected:** HTTP 200 (or appropriate status for your app)
</step>

</process>

<rollback>
**If deployment fails:**

1. Check logs for error cause
2. For simple rollback, redeploy previous commit:
   ```bash
   curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{"git_commit_sha": "previous-commit-sha"}'

   curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/deploy" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}"
   ```

3. For Docker image apps, specify previous tag:
   ```bash
   curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{"docker_registry_image_tag": "previous-tag"}'
   ```
</rollback>

<success_criteria>
Deployment is complete when:
- [ ] Target application(s) identified
- [ ] Pre-deployment checks passed
- [ ] Deploy command executed successfully
- [ ] Deployment status shows "running"
- [ ] Application responds at FQDN (if applicable)
- [ ] User confirms expected behavior
</success_criteria>
