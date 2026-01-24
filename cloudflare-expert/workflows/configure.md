<required_reading>
**Read reference files based on what's being configured:**
- references/dns.md - DNS record management
- references/security.md - Security features
- references/performance.md - Caching and optimization
- references/workers-ecosystem.md - Workers, Pages, R2, D1
- references/zero-trust.md - Access, Tunnel, Gateway
</required_reading>

<process>

<step name="identify_service">
**Step 1: Identify What to Configure**

Categories:
1. **DNS** - Add/edit/delete records
2. **SSL/TLS** - Certificate settings, encryption mode
3. **Security** - Firewall rules, WAF, rate limiting
4. **Performance** - Caching, Page Rules, optimization
5. **Workers** - Deploy serverless functions
6. **Pages** - Deploy static sites
7. **R2** - Object storage buckets
8. **D1** - SQLite databases
9. **Zero Trust** - Access policies, Tunnels

Ask user what they want to configure if not specified.
</step>

<step name="dns_config">
**Step 2a: DNS Configuration**

```bash
# List current records
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | \
  jq '.result[] | {id, type, name, content, proxied}'

# Create A record
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "A",
    "name": "subdomain",
    "content": "192.0.2.1",
    "ttl": 1,
    "proxied": true
  }'

# Create CNAME record
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "CNAME",
    "name": "www",
    "content": "domain.com",
    "ttl": 1,
    "proxied": true
  }'

# Update record
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"content": "new-ip-address"}'

# Delete record (DESTRUCTIVE - confirm first)
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</step>

<step name="ssl_config">
**Step 2b: SSL/TLS Configuration**

```bash
# Set SSL mode (off, flexible, full, strict)
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"strict"}'

# Enable Always Use HTTPS
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/always_use_https" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"on"}'

# Enable Automatic HTTPS Rewrites
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/automatic_https_rewrites" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"on"}'

# Set minimum TLS version
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/min_tls_version" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"1.2"}'
```
</step>

<step name="security_config">
**Step 2c: Security Configuration**

```bash
# Set security level (essentially_off, low, medium, high, under_attack)
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/security_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"medium"}'

# Create firewall rule (block country example)
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/firewall/rules" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '[{
    "filter": {
      "expression": "(ip.geoip.country eq \"CN\")"
    },
    "action": "block",
    "description": "Block China"
  }]'

# Create rate limit rule
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/rate_limits" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{
    "threshold": 100,
    "period": 60,
    "match": {"request": {"url_pattern": "/*"}},
    "action": {"mode": "challenge"}
  }'
```
</step>

<step name="cache_config">
**Step 2d: Caching Configuration**

```bash
# Set cache level (bypass, basic, simplified, aggressive)
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/cache_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"aggressive"}'

# Set browser cache TTL
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/browser_cache_ttl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":14400}'

# Enable minification
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/minify" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":{"css":"on","html":"on","js":"on"}}'

# Purge cache
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"purge_everything":true}'
```
</step>

<step name="workers_config">
**Step 2e: Workers Configuration**

```bash
# List Workers
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/workers/scripts" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Deploy a Worker script
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/workers/scripts/my-worker" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/javascript" \
  --data 'addEventListener("fetch", event => { event.respondWith(new Response("Hello!")) })'

# Create Worker route
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/workers/routes" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"pattern": "example.com/api/*", "script": "my-worker"}'
```
</step>

<step name="r2_config">
**Step 2f: R2 Storage Configuration**

```bash
# List R2 buckets
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/r2/buckets" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Create R2 bucket
curl -X POST "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/r2/buckets" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"name": "my-bucket"}'
```
</step>

</process>

<success_criteria>
Configuration is complete when:
- [ ] Service identified
- [ ] Current settings reviewed
- [ ] New configuration applied
- [ ] API returns success
- [ ] Settings verified
</success_criteria>
