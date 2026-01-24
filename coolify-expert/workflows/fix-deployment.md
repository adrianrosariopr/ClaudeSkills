<required_reading>
**Read these reference files:**
- references/troubleshooting.md - Common failures and fixes
- references/api-endpoints.md - For API operations
</required_reading>

<process>

<step name="identify_failure">
**Step 1: Identify the Failing Resource**

If user specified a resource, get its details:

```bash
# Get application details by UUID
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'
```

If no resource specified, run diagnose workflow first to find failures.

Key fields to examine:
- `status` - Current state (failed, exited, restarting)
- `docker_compose_location` - For compose-based apps
- `dockerfile_location` - For Dockerfile builds
- `build_pack` - Build method (nixpacks, dockerfile, compose)
</step>

<step name="get_logs">
**Step 2: Retrieve Deployment Logs**

```bash
# Get recent deployment logs
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}/logs" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq -r '.logs // .'
```

**Common error patterns to look for:**

| Log Pattern | Likely Cause | Fix |
|-------------|--------------|-----|
| `ECONNREFUSED` | Service dependency down | Check dependent services |
| `permission denied` | File/volume permissions | Fix ownership, check mount paths |
| `OOMKilled` | Out of memory | Increase memory limit |
| `port already in use` | Port conflict | Change exposed port |
| `image not found` | Registry auth or image missing | Check registry credentials |
| `npm ERR!` | Node build failure | Check package.json, node version |
| `build failed` | Dockerfile/Nixpacks issue | Review build config |
</step>

<step name="diagnose_root_cause">
**Step 3: Diagnose Root Cause**

Based on logs and status, categorize the failure:

**Build Failures:**
- Missing dependencies in Dockerfile/nixpacks
- Invalid package.json or requirements.txt
- Network issues during build (npm registry, pip, etc.)

**Runtime Failures:**
- Environment variables missing
- Database connection issues
- Port binding conflicts
- Volume mount problems
- Health check failures

**Infrastructure Failures:**
- Server disk full
- Docker daemon issues
- Network connectivity
- SSL certificate problems
</step>

<step name="apply_fix">
**Step 4: Apply the Fix**

**For environment variable issues:**
```bash
# Get current env vars
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.environment_variables'

# Update environment variables (PATCH)
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"environment_variables": [{"key": "VAR_NAME", "value": "value"}]}'
```

**For restart/redeploy:**
```bash
# Restart (keeps current container)
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/restart" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Redeploy (full rebuild)
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/deploy" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"

# Force deploy (ignore cache)
curl -X POST "${COOLIFY_URL}/api/v1/applications/{uuid}/deploy?force=true" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**For configuration changes:**
```bash
# Update application config
curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "memory_limit": "512M",
    "cpu_limit": "1",
    "health_check_enabled": false
  }'
```
</step>

<step name="verify_fix">
**Step 5: Verify the Fix**

After applying fix, monitor deployment:

```bash
# Check application status
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '{status, fqdn}'

# Poll until deployment completes (run multiple times)
watch -n 5 'curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq .status'
```

**Success indicators:**
- Status changes to "running"
- Health check passes (if enabled)
- No new errors in logs
- Application responds at FQDN
</step>

</process>

<escalation>
**If fix doesn't work:**

1. Check server resources: `curl -s "${COOLIFY_URL}/api/v1/servers" -H "Authorization: Bearer ${COOLIFY_API_KEY}"`
2. Review Docker daemon logs on the server
3. Check if other apps on same server are affected
4. Consider restarting Docker/server as last resort
5. Ask user for SSH access to investigate directly
</escalation>

<success_criteria>
Fix is complete when:
- [ ] Root cause identified
- [ ] Fix applied (config change, restart, redeploy)
- [ ] Application status shows "running"
- [ ] No errors in recent logs
- [ ] User confirms application is working
</success_criteria>
