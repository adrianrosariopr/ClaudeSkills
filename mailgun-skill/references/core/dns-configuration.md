<overview>
DNS configuration is the foundation of email deliverability. Proper SPF, DKIM, and DMARC records authenticate your emails and prevent them from landing in spam.
</overview>

<spf_configuration>
## SPF (Sender Policy Framework)

Specifies which servers can send email for your domain.

<record_format>
**Basic SPF for Mailgun:**
```
Type: TXT
Host: @ (or your subdomain)
Value: v=spf1 include:mailgun.org ~all
```

**Merging with existing SPF:**
```
v=spf1 ip4:your.server.ip include:_spf.google.com include:mailgun.org ~all
```

**Qualifiers:**
- `~all` (softfail) - Accept but mark suspicious (recommended)
- `-all` (hardfail) - Reject non-matching senders
- `?all` (neutral) - No opinion
</record_format>

<spf_limits>
**10 DNS Lookup Limit:**
Each `include:`, `a:`, `mx:`, and `redirect:` counts as one lookup.

**Check your lookup count:**
```bash
# Online tools
https://dmarcian.com/spf-survey/

# Or dig manually
dig TXT yourdomain.com +short
```

**If exceeding 10 lookups:**
- Use `ip4:` or `ip6:` instead of `include:` where possible
- Use SPF flattening services
- Split email sending across subdomains
</spf_limits>
</spf_configuration>

<dkim_configuration>
## DKIM (DomainKeys Identified Mail)

Cryptographically signs outgoing emails to prove authenticity.

<record_format>
**Mailgun provides the record:**
```
Type: TXT (or CNAME)
Host: selector._domainkey.mail (varies by domain)
Value: k=rsa; p=MIGfMA0GC... (long key string)
```

**Key sizes:**
- 1024-bit (older default)
- 2048-bit (current default, more secure)

**Note:** 2048-bit keys may need to be split across multiple strings in some DNS providers.
</record_format>

<verification>
**Verify DKIM:**
```bash
dig TXT selector._domainkey.yourdomain.com +short
```

Should return the public key.
</verification>
</dkim_configuration>

<dmarc_configuration>
## DMARC (Domain-based Message Authentication)

Tells receivers how to handle emails that fail SPF/DKIM.

<record_format>
**Monitoring mode (start here):**
```
Type: TXT
Host: _dmarc
Value: v=DMARC1; p=none; rua=mailto:dmarc-reports@yourdomain.com
```

**Quarantine mode:**
```
v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc-reports@yourdomain.com
```

**Reject mode (strictest):**
```
v=DMARC1; p=reject; rua=mailto:dmarc-reports@yourdomain.com
```
</record_format>

<dmarc_tags>
| Tag | Meaning | Example |
|-----|---------|---------|
| `p` | Policy (none/quarantine/reject) | `p=quarantine` |
| `pct` | Percentage to apply policy | `pct=50` |
| `rua` | Aggregate report email | `rua=mailto:dmarc@example.com` |
| `ruf` | Forensic report email | `ruf=mailto:dmarc@example.com` |
| `sp` | Subdomain policy | `sp=reject` |
| `adkim` | DKIM alignment (r=relaxed, s=strict) | `adkim=r` |
| `aspf` | SPF alignment | `aspf=r` |
</dmarc_tags>

<dmarc_rollout>
**Recommended rollout:**
1. Week 1-2: `p=none` (monitor only)
2. Week 3-4: `p=quarantine; pct=10` (10% quarantined)
3. Week 5-6: `p=quarantine; pct=50`
4. Week 7+: `p=quarantine; pct=100` or `p=reject`
</dmarc_rollout>
</dmarc_configuration>

<mx_records>
## MX Records (Optional - For Receiving)

Only needed if you want to receive email at your Mailgun domain.

```
Type: MX
Host: mail (your subdomain)
Priority: 10
Value: mxa.mailgun.org

Type: MX
Host: mail
Priority: 10
Value: mxb.mailgun.org
```
</mx_records>

<tracking_cname>
## Tracking CNAME

Required for open/click tracking to work properly.

```
Type: CNAME
Host: email.mail (Mailgun provides exact value)
Value: mailgun.org
```
</tracking_cname>

<cloudflare_warning>
## Cloudflare Users

**Critical:** Disable proxy (orange cloud) for all Mailgun DNS records.

Email-related records must resolve directly, not through Cloudflare's proxy:
- SPF TXT record
- DKIM TXT record
- MX records
- Tracking CNAME

Set to "DNS only" (gray cloud) for these records.
</cloudflare_warning>

<verification_commands>
## Verification Commands

```bash
# Check SPF
dig TXT yourdomain.com +short | grep spf

# Check DKIM
dig TXT selector._domainkey.yourdomain.com +short

# Check DMARC
dig TXT _dmarc.yourdomain.com +short

# Check MX
dig MX mail.yourdomain.com +short

# Online verification
https://mxtoolbox.com/SuperTool.aspx
```
</verification_commands>

<propagation>
## DNS Propagation

Changes take 24-48 hours to propagate globally.

**Speed up verification:**
- Lower TTL before making changes
- Use DNS providers with fast propagation (Cloudflare)
- Check propagation: https://www.whatsmydns.net/
</propagation>
