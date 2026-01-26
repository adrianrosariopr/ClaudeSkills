<required_reading>
**Read these reference files as needed:**
- references/api-endpoints.md - For management endpoints
</required_reading>

<process>

<step name="identify_action">
**Step 1: Identify Management Action**

Determine the action:
- **Start** - Start a stopped application/service
- **Stop** - Stop a running application/service
- **Restart** - Restart without redeploying
- **Env Vars** - View or modify environment variables
- **Scale** - Adjust replicas or resources
- **Config** - Modify application configuration
</step>

<step name="lifecycle_operations">
**Step 2: Lifecycle Operations**

**Start Application:**
```bash
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/start" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**Stop Application:**
```bash
# WARNING: This stops the running container
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/stop" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**Restart Application:**
```bash
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/restart" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**For Services (pre-built stacks like WordPress, n8n):**
```bash
# Start service
curl -X POST "${COOLIFY_URL}/api/v1/services/{uuid}/start" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Stop service
curl -X POST "${COOLIFY_URL}/api/v1/services/{uuid}/stop" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Restart service
curl -X POST "${COOLIFY_URL}/api/v1/services/{uuid}/restart" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**For Databases:**
```bash
# Start database
curl -X POST "${COOLIFY_URL}/api/v1/databases/{uuid}/start" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Stop database
curl -X POST "${COOLIFY_URL}/api/v1/databases/{uuid}/stop" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Restart database
curl -X POST "${COOLIFY_URL}/api/v1/databases/{uuid}/restart" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```
</step>

<step name="environment_variables">
**Step 3: Environment Variables**

**View current env vars:**
```bash
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '.environment_variables'
```

**Add/Update environment variable:**
```bash
# Get current envs, add new one, update
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "environment_variables_preview": [
      {"key": "NEW_VAR", "value": "new_value", "is_build_time": false}
    ]
  }'
```

**Note:** After changing env vars, restart or redeploy for changes to take effect:
```bash
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/restart" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```
</step>

<step name="resource_config">
**Step 4: Resource Configuration**

**Update memory/CPU limits:**
```bash
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "limits_memory": "512M",
    "limits_cpus": "1"
  }'
```

**Update health check settings:**
```bash
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "health_check_enabled": true,
    "health_check_path": "/health",
    "health_check_port": "3000",
    "health_check_interval": 30,
    "health_check_timeout": 10,
    "health_check_retries": 3
  }'
```
</step>

<step name="batch_operations">
**Step 5: Batch Operations**

**Restart all applications in a project:**
```bash
PROJECT_UUID="your-project-uuid"

curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq --arg proj "$PROJECT_UUID" '.[] | select(.project_uuid == $proj) | .uuid' -r | \
  while read uuid; do
    echo "Restarting: $uuid"
    curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/restart" \
      -H "Authorization: Bearer ${COOLIFY_API_KEY}"
    sleep 2
  done
```

**Stop all applications (maintenance mode):**
```bash
# WARNING: This stops ALL applications
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '.[] | .uuid' -r | \
  while read uuid; do
    echo "Stopping: $uuid"
    curl -X POST "${COOLIFY_URL}/api/v1/applications/${uuid}/stop" \
      -H "Authorization: Bearer ${COOLIFY_API_KEY}"
  done
```
</step>

<step name="verify_action">
**Step 6: Verify Action**

After any management action, verify the result:

```bash
# Check status
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '{name, status, updated_at}'
```

**Expected statuses:**
- After start: `running`
- After stop: `exited` or `stopped`
- After restart: `running` (brief `restarting` state)
</step>

</process>

<safety_reminder>
**Before batch operations:**
1. List all affected resources
2. Confirm with user before executing
3. For production, consider doing one at a time
4. Have rollback plan ready
</safety_reminder>

<success_criteria>
Management action is complete when:
- [ ] Target resource(s) identified
- [ ] Action executed without API errors
- [ ] Resource status matches expected state
- [ ] User confirms expected behavior
</success_criteria>
