<overview>
Common Coolify deployment failures and their solutions. Use this reference when diagnosing and fixing broken deployments.
</overview>

<build_failures>

<failure name="nixpacks_build_failed">
**Symptoms:**
- Build fails during Nixpacks phase
- Error mentions missing dependencies or unsupported runtime

**Common causes:**
- Unsupported language version
- Missing `package.json`, `requirements.txt`, or equivalent
- Private dependencies without auth

**Solutions:**
1. Check Nixpacks supports your runtime version
2. Ensure dependency file is in repo root
3. For private deps, add registry auth to env vars
4. Switch to Dockerfile build for more control:
   ```bash
   curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{"build_pack": "dockerfile"}'
   ```
</failure>

<failure name="dockerfile_build_failed">
**Symptoms:**
- Build fails during Docker build
- Error in Dockerfile instructions

**Common causes:**
- Syntax error in Dockerfile
- Base image not found
- COPY/ADD source file missing
- Build arg not provided

**Solutions:**
1. Validate Dockerfile syntax locally: `docker build .`
2. Check base image exists and is accessible
3. Ensure COPY sources match repo structure
4. Add missing build args to environment variables
</failure>

<failure name="npm_install_failed">
**Symptoms:**
- `npm ERR!` in build logs
- Package resolution failures

**Common causes:**
- Node version mismatch
- Private npm registry auth
- Package conflicts
- Network issues to npm registry

**Solutions:**
1. Add `.nvmrc` or `engines` in package.json to specify Node version
2. For private registry: set `NPM_TOKEN` env var
3. Clear npm cache with force deploy: `?force=true`
4. Check package-lock.json is committed
</failure>

<failure name="python_install_failed">
**Symptoms:**
- `pip install` failures
- Missing system dependencies

**Common causes:**
- Missing `requirements.txt`
- System library dependencies (libpq, etc.)
- Python version mismatch

**Solutions:**
1. Ensure `requirements.txt` in repo root
2. For system deps, use Dockerfile with apt-get
3. Add `runtime.txt` with Python version
</failure>

</build_failures>

<runtime_failures>

<failure name="container_crash_loop">
**Symptoms:**
- Status shows `restarting` repeatedly
- Container exits immediately after starting

**Common causes:**
- Application error on startup
- Missing environment variables
- Port binding issues
- Health check failing

**Solutions:**
1. Check logs for startup error:
   ```bash
   curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}/logs" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}"
   ```
2. Verify all required env vars are set
3. Check `PORT` env var matches app's listen port
4. Disable health check temporarily:
   ```bash
   curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{"health_check_enabled": false}'
   ```
</failure>

<failure name="oom_killed">
**Symptoms:**
- Container suddenly stops
- `OOMKilled` in logs or Docker events

**Common causes:**
- Memory limit too low
- Memory leak in application
- Large file processing

**Solutions:**
1. Increase memory limit:
   ```bash
   curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{"limits_memory": "1G"}'
   ```
2. Profile application memory usage
3. Add swap if server supports it
</failure>

<failure name="port_conflict">
**Symptoms:**
- `port already in use` error
- Container fails to start

**Common causes:**
- Another container using same port
- Host port conflict
- Previous container not fully stopped

**Solutions:**
1. Change exposed port in Coolify settings
2. Stop conflicting container
3. Use dynamic port mapping (let Coolify assign)
</failure>

<failure name="database_connection_refused">
**Symptoms:**
- `ECONNREFUSED` to database
- App can't connect to database

**Common causes:**
- Database not running
- Wrong connection string
- Network/firewall issues
- Database in different Docker network

**Solutions:**
1. Check database is running:
   ```bash
   curl -s "${COOLIFY_URL}/api/v1/databases" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.[].status'
   ```
2. Verify connection string uses Docker hostname (service name)
3. Ensure app and database in same Coolify project
4. Check DATABASE_URL env var format
</failure>

<failure name="health_check_failed">
**Symptoms:**
- Container running but marked unhealthy
- Traffic not routed to container

**Common causes:**
- Health check path returns non-200
- Health check port wrong
- App slow to start (timeout)

**Solutions:**
1. Verify health check endpoint exists and returns 200
2. Check health check configuration:
   ```bash
   curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
     jq '{health_check_enabled, health_check_path, health_check_port}'
   ```
3. Increase timeout and start period:
   ```bash
   curl -X PATCH "${COOLIFY_URL}/api/v1/applications/{uuid}" \
     -H "Authorization: Bearer ${COOLIFY_API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{
       "health_check_timeout": 30,
       "health_check_start_period": 60,
       "health_check_retries": 5
     }'
   ```
</failure>

</runtime_failures>

<infrastructure_failures>

<failure name="server_unreachable">
**Symptoms:**
- `is_reachable: false` for server
- Deployments queue but don't run

**Common causes:**
- SSH connection failed
- Server down
- Firewall blocking Coolify

**Solutions:**
1. Verify server is up (ping, SSH manually)
2. Check SSH key is valid
3. Verify firewall allows Coolify IP
4. Validate server in Coolify dashboard
</failure>

<failure name="disk_full">
**Symptoms:**
- Builds fail with `no space left on device`
- Containers can't write

**Common causes:**
- Docker images accumulating
- Log files growing
- Build cache full

**Solutions:**
1. SSH to server and clean Docker:
   ```bash
   docker system prune -af
   docker volume prune -f
   ```
2. Remove old images
3. Check application log rotation
4. Expand disk if needed
</failure>

<failure name="docker_daemon_issues">
**Symptoms:**
- All operations fail on server
- `Cannot connect to Docker daemon`

**Common causes:**
- Docker service stopped
- Docker socket permissions
- Docker daemon crash

**Solutions:**
1. SSH to server and restart Docker:
   ```bash
   sudo systemctl restart docker
   ```
2. Check Docker status: `sudo systemctl status docker`
3. Review Docker logs: `sudo journalctl -u docker`
</failure>

</infrastructure_failures>

<diagnostic_commands>
**Get full application state:**
```bash
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '.'
```

**Get recent logs:**
```bash
curl -s "${COOLIFY_URL}/api/v1/applications/{uuid}/logs?tail=100" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}"
```

**Check server health:**
```bash
curl -s "${COOLIFY_URL}/api/v1/servers/{server-uuid}" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | jq '{is_reachable, is_usable}'
```

**List all failing resources:**
```bash
curl -s "${COOLIFY_URL}/api/v1/applications" \
  -H "Authorization: Bearer ${COOLIFY_API_KEY}" | \
  jq '[.[] | select(.status != "running")] | .[] | {name, uuid, status}'
```
</diagnostic_commands>
