<required_reading>
**Read these reference files as needed:**
- references/dns.md - For DNS-specific checks
- references/security.md - For SSL/security checks
- references/common-issues.md - For interpreting error states
</required_reading>

<process>

<step name="list_all_zones">
**Step 1: List All Zones**

```bash
curl -s "https://api.cloudflare.com/client/v4/zones" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | \
  jq '.result[] | {name, id, status, paused, plan: .plan.name}'
```

Check for:
- `status: "active"` - Zone is working
- `status: "pending"` - Nameservers not updated
- `paused: true` - Zone is paused (traffic bypasses Cloudflare)
</step>

<step name="check_dns_health">
**Step 2: Check DNS Health Per Zone**

For each zone, verify DNS records exist and are proxied:

```bash
ZONE_ID="zone-id-here"
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | \
  jq '.result[] | {type, name, content, proxied, ttl}'
```

Key checks:
- Root domain (@) has A or CNAME record
- www subdomain exists
- MX records for email
- `proxied: true` for web traffic (orange cloud)
</step>

<step name="check_ssl_status">
**Step 3: Check SSL/TLS Status**

```bash
# SSL mode
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Certificate status
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/ssl/certificate_packs" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | \
  jq '.result[] | {type, status, hosts}'
```

SSL modes:
- `off` - No HTTPS (bad)
- `flexible` - HTTPS to Cloudflare only (ok for static)
- `full` - HTTPS end-to-end (origin can self-sign)
- `strict` - HTTPS with valid origin cert (best)
</step>

<step name="check_security_settings">
**Step 4: Check Security Settings**

```bash
# Security level
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/security_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# WAF status (if available)
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/waf" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Firewall rules count
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/firewall/rules" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result | length'
```
</step>

<step name="check_performance">
**Step 5: Check Performance Settings**

```bash
# Caching level
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/cache_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Minification
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/minify" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Browser cache TTL
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/browser_cache_ttl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'
```
</step>

<step name="check_workers">
**Step 6: Check Workers/Pages (Account Level)**

```bash
# List Workers
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/workers/scripts" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result[] | {id, modified_on}'

# List Pages projects
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/pages/projects" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result[] | {name, subdomain, domains}'

# List R2 buckets
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/r2/buckets" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'
```
</step>

</process>

<output_format>
Present findings in this format:

**Cloudflare Health Report**

**Zones:** X total, Y active, Z pending

| Zone | Status | SSL | Security | Issues |
|------|--------|-----|----------|--------|
| domain.com | active | strict | medium | None |
| other.com | active | flexible | low | SSL should be strict |

**Developer Platform:**
- Workers: X scripts
- Pages: X projects
- R2: X buckets
- D1: X databases

**Issues Found:**
1. [Zone] - [Issue] - [Recommendation]
2. ...

**Recommendations:**
1. ...
</output_format>

<success_criteria>
Diagnosis is complete when:
- [ ] All zones listed with status
- [ ] DNS records checked for each zone
- [ ] SSL/TLS status verified
- [ ] Security settings reviewed
- [ ] Performance settings checked
- [ ] Workers/Pages/R2 inventory taken
- [ ] Issues identified and reported
- [ ] Recommendations provided
</success_criteria>
