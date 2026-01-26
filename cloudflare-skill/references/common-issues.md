<overview>
Common Cloudflare issues, error codes, and their solutions.
</overview>

<error_codes>

<error code="520">
**Web server returned an unknown error**

**Cause:** Origin returned empty, invalid, or unexpected response.

**Fixes:**
1. Check origin server is running
2. Check origin application logs
3. Verify origin responds to requests
4. Check for firewall blocking Cloudflare IPs
</error>

<error code="521">
**Web server is down**

**Cause:** Origin refused the connection.

**Fixes:**
1. Verify origin server is running
2. Check origin firewall allows Cloudflare IPs
3. Verify correct port (80/443)
4. Check origin IP in DNS is correct
</error>

<error code="522">
**Connection timed out**

**Cause:** TCP handshake timed out (origin too slow or unreachable).

**Fixes:**
1. Check origin server load
2. Verify firewall allows Cloudflare IPs
3. Check for network issues between CF and origin
4. Increase origin timeout if possible
</error>

<error code="523">
**Origin is unreachable**

**Cause:** Cloudflare cannot reach origin IP.

**Fixes:**
1. Verify DNS points to correct IP
2. Check origin server is online
3. Check network routing to origin
</error>

<error code="524">
**A timeout occurred**

**Cause:** Origin took longer than 100 seconds to respond.

**Fixes:**
1. Optimize slow backend endpoints
2. Increase timeout (Enterprise only)
3. Use Workers to handle long requests
4. Implement async processing for slow operations
</error>

<error code="525">
**SSL handshake failed**

**Cause:** Cloudflare couldn't establish SSL connection to origin.

**Fixes:**
1. Verify origin supports HTTPS
2. Check origin SSL certificate is valid
3. Use "Full" mode instead of "Strict" if self-signed
4. Verify origin listens on port 443
</error>

<error code="526">
**Invalid SSL certificate**

**Cause:** Origin certificate invalid, expired, or doesn't match domain.

**Fixes:**
1. Install valid SSL certificate on origin
2. Use Cloudflare Origin Certificate (free)
3. Switch from "Strict" to "Full" mode temporarily
4. Check certificate covers the domain
</error>

<error code="527">
**Railgun Listener error**

**Cause:** Railgun connection issue (Enterprise feature).

**Fixes:**
1. Check Railgun Listener is running
2. Verify Railgun configuration
</error>

<error code="530">
**Origin DNS error (with 1xxx)**

**Cause:** Combined error - check the 1xxx code for details.
</error>

</error_codes>

<dns_issues>

<issue name="zone_pending">
**Zone status: Pending**

**Cause:** Nameservers not updated at registrar.

**Fix:**
1. Go to registrar (GoDaddy, Namecheap, etc.)
2. Find nameserver settings
3. Replace with Cloudflare's assigned nameservers
4. Wait up to 24 hours for propagation
</issue>

<issue name="dns_not_resolving">
**DNS record not resolving**

**Causes:**
- Record doesn't exist
- Wrong record type
- TTL propagation delay

**Fix:**
1. Verify record exists in Cloudflare dashboard
2. Check record type matches need (A vs CNAME)
3. Wait for TTL to expire (or lower TTL first)
4. Flush local DNS: `sudo dscacheutil -flushcache` (Mac)
</issue>

<issue name="too_many_redirects">
**ERR_TOO_MANY_REDIRECTS**

**Cause:** Redirect loop between Cloudflare and origin.

**Common scenario:**
- SSL mode: Flexible
- Origin forces HTTPS redirect
- CF sends HTTP → Origin redirects to HTTPS → CF sends HTTP → loop

**Fix:**
1. Set SSL mode to "Full" or "Strict"
2. Or disable HTTPS redirect on origin
</issue>

</dns_issues>

<ssl_issues>

<issue name="mixed_content">
**Mixed content warnings**

**Cause:** HTTPS page loading HTTP resources.

**Fixes:**
1. Enable "Automatic HTTPS Rewrites"
2. Fix source code to use HTTPS URLs
3. Use protocol-relative URLs (`//example.com`)
</issue>

<issue name="certificate_not_issued">
**Universal SSL not issued**

**Causes:**
- Zone too new (wait 15-30 min)
- CAA records blocking issuance
- DNS not properly configured

**Fixes:**
1. Wait up to 24 hours for new zones
2. Check CAA records allow Cloudflare CAs
3. Verify DNS is proxied (orange cloud)
</issue>

</ssl_issues>

<performance_issues>

<issue name="not_caching">
**Content not being cached**

**Causes:**
- Cache-Control headers from origin
- Cookies being set
- Query strings
- POST requests (never cached)

**Debugging:**
```bash
curl -sI https://example.com/file | grep -i "cf-cache-status"
```

**CF-Cache-Status values:**
- `HIT` - Served from cache
- `MISS` - Fetched from origin, now cached
- `BYPASS` - Not cached (check headers)
- `DYNAMIC` - Not eligible for caching
- `EXPIRED` - Was cached, now stale

**Fixes:**
1. Check origin Cache-Control headers
2. Create Cache Rules to override
3. Use "Cache Everything" page rule
</issue>

<issue name="slow_ttfb">
**Slow Time to First Byte**

**Causes:**
- Origin slow to respond
- No caching
- Far from edge location

**Fixes:**
1. Enable caching for static content
2. Optimize origin response time
3. Consider Argo Smart Routing (paid)
4. Use Workers for edge-side logic
</issue>

</performance_issues>

<security_issues>

<issue name="false_positive_blocks">
**Legitimate users being blocked**

**Debugging:**
1. Check Firewall Events in dashboard
2. Look for rule that triggered block
3. Check Security Level setting

**Fixes:**
1. Lower Security Level
2. Adjust or disable offending rule
3. Add IP/ASN to allowlist
4. Whitelist specific user agents
</issue>

<issue name="under_attack">
**Site under DDoS attack**

**Immediate actions:**
1. Enable "I'm Under Attack" mode
2. Add rate limiting rules
3. Block attacking countries/ASNs
4. Enable Bot Fight Mode
5. Consider enabling Waiting Room
</issue>

</security_issues>

<workers_issues>

<issue name="worker_not_triggering">
**Worker not running on route**

**Causes:**
- Route pattern doesn't match
- Worker not deployed
- Route not associated with zone

**Fixes:**
1. Verify route pattern matches URL
2. Check worker is deployed and active
3. Verify route is in correct zone
</issue>

<issue name="worker_errors">
**Worker throwing errors**

**Debugging:**
1. Check Workers logs in dashboard
2. Use `wrangler tail` for real-time logs
3. Add try-catch for error handling

**Common causes:**
- Syntax errors
- Missing environment bindings
- Exceeding CPU/memory limits
</issue>

</workers_issues>

<quick_diagnostics>
```bash
# Check zone status
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '{status, name_servers}'

# Check SSL settings
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/ssl" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Check security level
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/security_level" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Check HTTP response headers
curl -sI https://example.com | grep -E "(cf-|server|cache)"
```
</quick_diagnostics>
