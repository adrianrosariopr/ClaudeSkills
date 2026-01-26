<required_reading>
**Read these reference files as needed:**
- references/api-endpoints.md - For endpoint details
- references/troubleshooting.md - For interpreting error states
</required_reading>

<process>

<step name="verify_connection">
**Step 1: Verify API Connection**

First, check that credentials are configured and the API is reachable:

```bash
# Check environment variables
echo "COOLIFY_URL: ${COOLIFY_URL:-NOT SET}"
echo "COOLIFY_API_KEY: ${COOLIFY_API_KEY:+SET (hidden)}"

# Test API connection
curl -s -w "\nHTTP Status: %{http_code}\n" \
  "${COOLIFY_URL}/api/v1/version" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**Expected:** HTTP 200 with version info like `{"version":"4.0.0-beta.462"}`

**If connection fails:**
- 401: Invalid API key - regenerate at `${COOLIFY_URL}/security/api-tokens`
- Connection refused: Check URL and ensure Coolify is running
- SSL error: Verify HTTPS certificate or use `--insecure` for self-signed
</step>

<step name="list_projects">
**Step 2: List All Projects**

```bash
curl -s "${COOLIFY_URL}/api/v1/projects" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'
```

This returns all projects with their UUIDs. Note the project UUIDs for further queries.
</step>

<step name="list_applications">
**Step 3: List All Applications**

```bash
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'
```

Look for applications with problematic statuses:
- `status: "exited"` - Container stopped
- `status: "restarting"` - Container crash loop
- `status: "failed"` - Deployment failed
</step>

<step name="list_services">
**Step 4: List Services and Databases**

```bash
# List services (pre-built stacks)
curl -s "${COOLIFY_URL}/api/v1/services" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'

# List databases
curl -s "${COOLIFY_URL}/api/v1/databases" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'
```
</step>

<step name="check_servers">
**Step 5: Check Server Health**

```bash
curl -s "${COOLIFY_URL}/api/v1/servers" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'
```

Check for:
- `is_reachable: false` - Server connection issues
- `is_usable: false` - Server not properly configured
- High resource usage indicators
</step>

<step name="identify_failures">
**Step 6: Identify Failing Resources**

Parse all resources and filter for failures:

```bash
# Applications with issues
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '[.[] | select(.status != "running" and .status != null)] |
      .[] | {name, uuid, status, fqdn}'
```

Report findings to user with:
- Resource name and UUID
- Current status
- FQDN (URL) if available
- Recommended action
</step>

</process>

<output_format>
Present findings in this format:

**Instance Health: [HEALTHY/DEGRADED/CRITICAL]**

**Projects:** X total
**Applications:** X running, Y failed, Z stopped
**Services:** X running, Y failed
**Databases:** X running, Y failed
**Servers:** X reachable, Y unreachable

**Issues Found:**
| Resource | Type | Status | Action |
|----------|------|--------|--------|
| app-name | Application | failed | Check logs |
| db-name | Database | exited | Restart |

**Recommended Next Steps:**
1. ...
2. ...
</output_format>

<success_criteria>
Diagnosis is complete when:
- [ ] API connection verified
- [ ] All projects listed
- [ ] All applications checked
- [ ] All services checked
- [ ] All databases checked
- [ ] Server health verified
- [ ] Failing resources identified and reported
- [ ] Recommended actions provided
</success_criteria>
