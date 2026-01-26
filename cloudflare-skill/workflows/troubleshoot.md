<required_reading>
**Read these reference files:**
- references/common-issues.md - Error codes and solutions
- references/dns.md - DNS troubleshooting
- references/security.md - SSL/security issues
</required_reading>

<process>

<step name="identify_issue">
**Step 1: Identify the Issue**

Common issue categories:
1. **DNS** - Records not resolving, propagation delays
2. **SSL/TLS** - Certificate errors, mixed content, 525/526 errors
3. **5xx Errors** - 502, 503, 520-527 (Cloudflare-specific)
4. **4xx Errors** - 403 blocked, 404 not found
5. **Performance** - Slow loading, cache not working
6. **Security** - False positives, blocked requests
7. **Workers** - Script errors, routing issues

Ask user which issue they're experiencing if not clear.
</step>

<step name="dns_troubleshooting">
**Step 2a: DNS Troubleshooting**

```bash
# Check if zone nameservers are correct
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | \
  jq '{status, name_servers}'

# Check DNS record exists
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records?name=subdomain.domain.com" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# External DNS check
dig +short domain.com A
dig +short domain.com @1.1.1.1 A
```

**Common DNS fixes:**
- Zone pending → Update nameservers at registrar
- Record missing → Create the record
- Wrong IP → Update the record
- Not proxied → Enable proxy (orange cloud)
</step>

<step name="ssl_troubleshooting">
**Step 2b: SSL/TLS Troubleshooting**

```bash
# Check SSL mode
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Check certificate status
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/ssl/certificate_packs" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Check Always Use HTTPS
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/always_use_https" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'
```

**SSL Error Codes:**
- **525** - SSL handshake failed → Origin doesn't support HTTPS or wrong port
- **526** - Invalid SSL certificate → Origin cert expired/invalid, use Full not Strict
- **Mixed content** - HTTP resources on HTTPS page → Enable Automatic HTTPS Rewrites
</step>

<step name="5xx_troubleshooting">
**Step 2c: 5xx Error Troubleshooting**

**Cloudflare-specific errors:**
- **520** - Web server returned empty response → Check origin is responding
- **521** - Origin refused connection → Origin down or blocking Cloudflare IPs
- **522** - Connection timed out → Origin overloaded or firewall blocking
- **523** - Origin unreachable → DNS points to wrong IP
- **524** - Timeout occurred → Origin taking too long (>100s)

```bash
# Check if origin is accessible (from your machine)
curl -sI https://origin-server-ip -H "Host: domain.com"

# Check Cloudflare is reaching origin
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/origin_error_page_pass_thru" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```

**Fixes:**
- 521/522 → Whitelist Cloudflare IPs at origin firewall
- 524 → Increase origin timeout or optimize backend
- 520 → Check origin application logs
</step>

<step name="security_troubleshooting">
**Step 2d: Security/Blocking Issues**

```bash
# Check security level
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/security_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# List firewall rules
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/firewall/rules" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Check rate limiting rules
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/rate_limits" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'
```

**If users getting blocked:**
- Lower security level temporarily
- Check firewall rules for overly broad blocks
- Review WAF managed rules
- Check IP Access Rules
</step>

<step name="cache_troubleshooting">
**Step 2e: Caching Issues**

```bash
# Check cache settings
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/cache_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Purge cache if needed
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"purge_everything":true}'

# Purge specific URLs
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"files":["https://domain.com/path/to/file"]}'
```

**Cache debugging:**
- Check `CF-Cache-Status` header in response
- `HIT` = served from cache
- `MISS` = fetched from origin
- `BYPASS` = cache bypassed (check Cache-Control headers)
</step>

<step name="apply_fix">
**Step 3: Apply the Fix**

Based on diagnosis, apply appropriate fix:

```bash
# Change SSL mode
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"full"}'

# Change security level
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/security_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"medium"}'

# Enable Always Use HTTPS
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/always_use_https" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"on"}'
```
</step>

<step name="verify_fix">
**Step 4: Verify the Fix**

```bash
# Re-check the setting
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/{setting}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Test the URL
curl -sI https://domain.com | head -20
```
</step>

</process>

<success_criteria>
Troubleshooting is complete when:
- [ ] Issue identified and categorized
- [ ] Root cause determined
- [ ] Fix applied
- [ ] Fix verified working
- [ ] User confirms issue resolved
</success_criteria>
