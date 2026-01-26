<overview>
Security features in Cloudflare: SSL/TLS, WAF, firewall rules, DDoS protection, and bot management.
</overview>

<ssl_tls>
<modes>
| Mode | Description | When to Use |
|------|-------------|-------------|
| Off | No HTTPS | Never (insecure) |
| Flexible | HTTPS to CF, HTTP to origin | Static sites without origin HTTPS |
| Full | HTTPS end-to-end, self-signed OK | Origin has self-signed cert |
| Full (Strict) | HTTPS, valid origin cert required | Production (recommended) |
</modes>

<settings>
**Always Use HTTPS:** Redirect all HTTP to HTTPS
**Automatic HTTPS Rewrites:** Fix mixed content by rewriting HTTP URLs
**Min TLS Version:** Set to 1.2 minimum (1.3 preferred for new sites)
**TLS 1.3:** Enable for better performance and security
**HSTS:** HTTP Strict Transport Security headers
</settings>

<origin_certificates>
Cloudflare can issue free origin certificates (15-year validity) for encrypting traffic between Cloudflare and your origin. Generate in dashboard or API.
</origin_certificates>

<api_commands>
```bash
# Get SSL mode
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Set SSL mode to strict
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
```
</api_commands>
</ssl_tls>

<waf>
**Web Application Firewall** protects against:
- SQL injection
- Cross-site scripting (XSS)
- Remote file inclusion
- Known vulnerabilities

**Managed Rulesets:**
- Cloudflare Managed Ruleset (free tier limited)
- OWASP Core Ruleset
- Cloudflare Exposed Credentials Check

**Custom Rules:** Create rules using Cloudflare's expression language.
</waf>

<firewall_rules>
<expression_examples>
```
# Block specific country
(ip.geoip.country eq "CN")

# Allow only specific IPs
(not ip.src in {192.0.2.0/24 198.51.100.0/24})

# Block specific user agent
(http.user_agent contains "BadBot")

# Protect admin area
(http.request.uri.path contains "/admin" and not ip.src in {192.0.2.1})

# Rate limit login
(http.request.uri.path eq "/login" and http.request.method eq "POST")

# Block known bad ASNs
(ip.geoip.asnum in {12345 67890})
```
</expression_examples>

<actions>
| Action | Description |
|--------|-------------|
| Block | Return 403 |
| Challenge | Show CAPTCHA |
| JS Challenge | JavaScript challenge |
| Managed Challenge | Smart challenge selection |
| Allow | Skip remaining rules |
| Log | Log but don't block |
</actions>

<api_commands>
```bash
# List firewall rules
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/firewall/rules" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Create firewall rule
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/firewall/rules" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '[{"filter":{"expression":"(ip.geoip.country eq \"CN\")"},"action":"block"}]'
```
</api_commands>
</firewall_rules>

<security_levels>
| Level | Description |
|-------|-------------|
| Essentially Off | Only challenges most threatening |
| Low | Challenges known threats |
| Medium | Challenges moderate threats (recommended) |
| High | Challenges all suspicious visitors |
| I'm Under Attack | Maximum protection, shows interstitial |
</security_levels>

<ddos_protection>
**Automatic DDoS protection** is always on and free:
- Layer 3/4 attacks (volumetric)
- Layer 7 attacks (HTTP floods)
- No configuration needed

**During attack:**
- Enable "I'm Under Attack" mode
- Tighten firewall rules
- Enable rate limiting
</ddos_protection>

<bot_management>
**Free tier:**
- Basic bot detection
- Challenge suspicious bots

**Paid features:**
- Bot score (0-99)
- Verified bot allowlist
- ML-based detection
- JA3 fingerprinting

**Turnstile** (free CAPTCHA alternative):
- Invisible challenges
- No user friction
- Easy integration
</bot_management>
