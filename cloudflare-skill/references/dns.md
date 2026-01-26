<overview>
DNS management in Cloudflare. Covers record types, proxy modes, and common configurations.
</overview>

<record_types>
| Type | Purpose | Proxied? | Example |
|------|---------|----------|---------|
| A | IPv4 address | Yes | `@ → 192.0.2.1` |
| AAAA | IPv6 address | Yes | `@ → 2001:db8::1` |
| CNAME | Alias to another domain | Yes | `www → example.com` |
| MX | Mail server | No | `@ → mail.example.com` |
| TXT | Text record (SPF, DKIM, verification) | No | `@ → v=spf1 ...` |
| SRV | Service location | No | `_sip._tcp → ...` |
| CAA | Certificate Authority Authorization | No | `@ → 0 issue "..."` |
| NS | Nameserver (managed by CF) | No | Auto |
</record_types>

<proxy_modes>
**Proxied (Orange Cloud):**
- Traffic flows through Cloudflare
- Gets CDN, caching, DDoS protection, WAF
- Hides origin IP
- Required for most Cloudflare features

**DNS Only (Gray Cloud):**
- Traffic goes directly to origin
- No Cloudflare features
- Use for: MX, non-HTTP services, when origin IP must be exposed

**When to use DNS Only:**
- Email (MX records)
- FTP servers
- SSH connections
- Non-HTTP/HTTPS services
- When troubleshooting SSL issues
</proxy_modes>

<common_configurations>

<config name="root_and_www">
**Root domain + www:**
```
@ (root)  →  A      →  192.0.2.1  (proxied)
www       →  CNAME  →  @          (proxied)
```
</config>

<config name="subdomain_to_different_server">
**Subdomain to different server:**
```
api       →  A      →  192.0.2.2  (proxied)
staging   →  A      →  192.0.2.3  (proxied)
```
</config>

<config name="email_setup">
**Email (Google Workspace):**
```
@         →  MX  →  aspmx.l.google.com       (priority 1)
@         →  MX  →  alt1.aspmx.l.google.com  (priority 5)
@         →  TXT →  v=spf1 include:_spf.google.com ~all
google._domainkey  →  TXT  →  [DKIM key]
```
</config>

<config name="pages_custom_domain">
**Cloudflare Pages custom domain:**
```
@         →  CNAME  →  project-name.pages.dev  (proxied)
www       →  CNAME  →  @                        (proxied)
```
</config>

<config name="tunnel">
**Cloudflare Tunnel:**
```
app       →  CNAME  →  tunnel-id.cfargotunnel.com  (proxied)
```
</config>

</common_configurations>

<api_operations>
```bash
# List all DNS records
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result'

# Create record
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"type":"A","name":"subdomain","content":"192.0.2.1","ttl":1,"proxied":true}'

# Update record
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"content":"new-ip"}'

# Delete record
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</api_operations>

<troubleshooting>
**Zone status "pending":**
- Nameservers not updated at registrar
- Check: `dig NS domain.com`
- Fix: Update nameservers to Cloudflare's assigned ones

**DNS not resolving:**
- Record doesn't exist
- Wrong record type
- TTL still propagating (wait up to TTL seconds)

**SSL errors after adding record:**
- Wait for certificate provisioning (up to 15 min)
- Check SSL mode matches origin capability
</troubleshooting>
