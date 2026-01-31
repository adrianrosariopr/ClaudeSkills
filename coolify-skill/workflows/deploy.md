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

<step name="check_deployment_source">
**Step 2: Check for Auto-Deploy (CRITICAL - Prevents Double Deployments)**

Before deploying via API, check if the app has automatic deployment configured:

```bash
# Check if app uses GitHub/GitLab App integration (auto-deploys on push)
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '{name, source_type, git_repository}'
```

**Auto-deploy sources (will deploy automatically on git push):**
- `source_type` contains `GithubApp` → GitHub App integration (auto-deploys)
- `source_type` contains `GitlabApp` → GitLab App integration (auto-deploys)
- `manual_webhook_secret_*` is set → Manual webhook configured

**Decision tree:**

| Scenario | Action |
|----------|--------|
| Git push just made + auto-deploy source | **SKIP API deploy** - webhook handles it |
| Git push just made + no auto-deploy | Use API deploy |
| No git push (redeploy existing) | Use API deploy |
| Force deploy requested | Use API deploy with `?force=true` |

**If user just committed and pushed code, and the app has an auto-deploy source, inform them:**
> "This app has automatic deployment via [GitHub/GitLab] App. The push you just made will trigger a deployment automatically. No API call needed."

**Only proceed to Step 3 if API deployment is needed.**
</step>

<step name="pre_deploy_check">
**Step 3: Pre-Deployment Checks**

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
**Step 4a: Deploy Single Application (API method)**

**Only use this if Step 2 determined API deployment is needed.**

```bash
# Standard deploy (use the /deploy endpoint with uuid param)
curl -s -X POST "${COOLIFY_URL}/api/v1/deploy?uuid={uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Force deploy (ignore build cache)
curl -s -X POST "${COOLIFY_URL}/api/v1/deploy?uuid={uuid}&force=true" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**Response contains deployment UUID for tracking.**
</step>

<step name="deploy_batch">
**Step 4b: Batch Deploy Multiple Applications**

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
**Step 4c: Deploy All Applications in Project**

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
**Step 5: Monitor Deployment Progress**

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
**Step 6: Post-Deployment Verification**

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
- [ ] Auto-deploy source checked (avoid double deployment)
- [ ] Pre-deployment checks passed (if API deploy needed)
- [ ] Deploy triggered (via webhook OR API, not both)
- [ ] Deployment status shows "running"
- [ ] Application responds at FQDN (if applicable)
- [ ] User confirms expected behavior
</success_criteria>
