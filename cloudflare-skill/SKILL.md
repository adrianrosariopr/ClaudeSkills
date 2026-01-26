---
name: cloudflare-skill
description: Expert Cloudflare administrator for managing DNS, security, performance, Workers, and Zero Trust across all domains. Use when working with Cloudflare zones, troubleshooting issues, configuring services, or choosing the right Cloudflare products for a project.
---

<essential_principles>

<authentication>
**Credentials:**

```
CF_EMAIL: Arosario@invisionnaire.com
CF_API_KEY: 996056aa77525d4f41f4cd46ef520900ef15f
CF_ACCOUNT_ID: 498725db75d32e48d269120f12bdf2e6
```

**Headers for all API calls:**
```
X-Auth-Email: Arosario@invisionnaire.com
X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f
```
</authentication>

<api_basics>
**Base URLs:**
- Zones: `https://api.cloudflare.com/client/v4/zones`
- Account: `https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6`
- User: `https://api.cloudflare.com/client/v4/user`

**Common curl pattern:**
```bash
curl -s "https://api.cloudflare.com/client/v4/zones" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```

**Response format:** All responses have `{success, errors, messages, result}` structure.
</api_basics>

<zones_are_dynamic>
Always fetch zones via API - never hardcode zone IDs. The user frequently adds new domains.

```bash
# List all zones
curl -s "https://api.cloudflare.com/client/v4/zones" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result[] | {name, id, status}'
```
</zones_are_dynamic>

<safety_guardrails>
**Before destructive operations:**
1. Confirm the target zone/resource matches expectations
2. For DNS changes, verify the record exists first
3. Never delete DNS records without explicit confirmation
4. For security rule changes, show current rules first

**Destructive Operations (require confirmation):**
- Deleting DNS records
- Purging entire cache
- Disabling security features
- Modifying SSL settings
- Deleting Workers/Pages
- Removing firewall rules
</safety_guardrails>

<service_categories>
**Core (every zone):**
- DNS management
- SSL/TLS certificates
- Caching & CDN
- DDoS protection (automatic)

**Security:**
- WAF (Web Application Firewall)
- Bot Management
- Page Shield
- Rate Limiting
- Firewall Rules

**Performance:**
- Page Rules
- Cache Rules
- Speed optimizations
- Argo Smart Routing (paid)
- Load Balancing (paid)

**Developer Platform:**
- Workers (serverless functions)
- Pages (static site hosting)
- KV (key-value storage)
- R2 (object storage, S3-compatible)
- D1 (SQLite database)
- Queues (message queues)
- Vectorize (vector database)
- AI (inference API)

**Zero Trust:**
- Access (identity-aware proxy)
- Tunnel (secure origin connection)
- Gateway (DNS filtering)
- WARP (VPN client)

**Media:**
- Stream (video hosting)
- Images (image optimization)

**Other:**
- Turnstile (CAPTCHA alternative)
- Zaraz (third-party tool manager)
- Web Analytics
- Waiting Room
</service_categories>

</essential_principles>

<intake>
What would you like to do?

1. **Diagnose** - Check zone health, DNS, SSL, security status across domains
2. **Troubleshoot** - Fix specific issues (DNS propagation, SSL errors, 5xx errors, etc.)
3. **Configure** - Set up or modify Cloudflare services
4. **Recommend** - Get advice on which Cloudflare services fit your project

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow | Description |
|----------|----------|-------------|
| 1, "diagnose", "health", "check", "status", "list" | workflows/diagnose.md | Health checks across all zones |
| 2, "troubleshoot", "fix", "error", "issue", "problem", "not working" | workflows/troubleshoot.md | Debug and fix specific issues |
| 3, "configure", "setup", "set up", "enable", "add", "create" | workflows/configure.md | Configure Cloudflare services |
| 4, "recommend", "suggest", "which", "should I use", "best" | workflows/recommend.md | Service recommendations |

**After reading the workflow, follow it exactly.**
</routing>

<quick_reference>
**Common Operations:**

```bash
# List all zones
curl -s "https://api.cloudflare.com/client/v4/zones" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result[] | {name, id}'

# Get zone details
curl -s "https://api.cloudflare.com/client/v4/zones/{zone_id}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# List DNS records for a zone
curl -s "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Purge cache for a zone
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"purge_everything":true}'

# Get SSL settings
curl -s "https://api.cloudflare.com/client/v4/zones/{zone_id}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</quick_reference>

<reference_index>
All domain knowledge in `references/`:

**DNS:** dns.md - Records, propagation, DNSSEC
**Security:** security.md - WAF, firewall, SSL/TLS, DDoS
**Performance:** performance.md - Caching, Page Rules, optimization
**Workers:** workers-ecosystem.md - Workers, Pages, KV, R2, D1, Queues
**Zero Trust:** zero-trust.md - Access, Tunnel, Gateway, WARP
**Troubleshooting:** common-issues.md - Error codes, fixes
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| diagnose.md | Health checks, zone status, DNS/SSL verification |
| troubleshoot.md | Debug and fix specific issues |
| configure.md | Set up and modify Cloudflare services |
| recommend.md | Service recommendations for projects |
</workflows_index>

<success_criteria>
Operations are successful when:
- API calls return `"success": true`
- DNS records resolve correctly
- SSL certificates are valid and active
- Security features show expected status
- User confirms the expected outcome
</success_criteria>
