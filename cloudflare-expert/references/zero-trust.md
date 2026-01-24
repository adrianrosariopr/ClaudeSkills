<overview>
Cloudflare Zero Trust: Access, Tunnel, Gateway, and WARP for secure access to internal resources.
</overview>

<access>
**Identity-aware proxy for applications.**

<features>
- Authenticate users before accessing apps
- Support for 15+ identity providers
- No VPN required
- Free for up to 50 users
</features>

<use_cases>
- Protect internal dashboards
- Secure staging environments
- Add SSO to any application
- Replace VPN for web apps
</use_cases>

<identity_providers>
- Google Workspace
- Microsoft Azure AD
- Okta
- GitHub
- OneLogin
- SAML 2.0
- OpenID Connect
- One-time PIN (email)
</identity_providers>

<setup_steps>
1. Go to Zero Trust dashboard
2. Add application (self-hosted or SaaS)
3. Configure identity provider
4. Create access policy (who can access)
5. Connect application via Tunnel or public hostname
</setup_steps>

<api_commands>
```bash
# List Access applications
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/access/apps" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# List Access policies
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/access/apps/${APP_ID}/policies" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</api_commands>
</access>

<tunnel>
**Secure connection from origin to Cloudflare without public IP.**

<features>
- No inbound firewall rules needed
- Outbound-only connection
- Encrypted by default
- Replaces traditional VPN for server access
</features>

<use_cases>
- Expose internal services securely
- Connect home lab to internet
- Protect origin server IP
- Connect Kubernetes services
</use_cases>

<setup_with_cloudflared>
```bash
# Install cloudflared
brew install cloudflared  # Mac
# or download from https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/

# Authenticate
cloudflared tunnel login

# Create tunnel
cloudflared tunnel create my-tunnel

# Configure tunnel (config.yml)
tunnel: <tunnel-id>
credentials-file: /path/to/credentials.json

ingress:
  - hostname: app.example.com
    service: http://localhost:3000
  - service: http_status:404

# Run tunnel
cloudflared tunnel run my-tunnel

# Create DNS record (auto)
cloudflared tunnel route dns my-tunnel app.example.com
```
</setup_with_cloudflared>

<api_commands>
```bash
# List tunnels
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/cfd_tunnel" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Get tunnel details
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/cfd_tunnel/${TUNNEL_ID}" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</api_commands>
</tunnel>

<gateway>
**Secure DNS and web filtering.**

<features>
- DNS filtering (block malware, adult content)
- HTTP/HTTPS inspection
- DLP (Data Loss Prevention)
- Shadow IT discovery
</features>

<dns_locations>
Set devices to use Gateway DNS:
- IPv4: 172.64.36.1, 172.64.36.2
- IPv6: 2606:4700:4700::1111
- DoH: https://dns.cloudflare.com/dns-query

Or use WARP client for automatic configuration.
</dns_locations>

<policies>
- Block malware domains
- Block phishing sites
- Block adult content
- Allow/block specific domains
- Block file types in downloads
</policies>
</gateway>

<warp>
**VPN client for devices.**

<features>
- Encrypts all device traffic
- Routes through Cloudflare network
- Faster than traditional VPNs
- Integrates with Gateway policies
</features>

<modes>
- **WARP** - Encrypt and optimize traffic
- **WARP+** - Add Argo routing for speed
- **Gateway with WARP** - Add DNS/HTTP filtering
- **Gateway with DoH** - DNS filtering only
</modes>

<deployment>
- Manual install from 1.1.1.1 app
- MDM deployment (Intune, Jamf, etc.)
- Managed via Zero Trust dashboard
</deployment>
</warp>

<device_posture>
**Check device security before granting access.**

<checks>
- OS version
- Disk encryption
- Firewall enabled
- Screen lock
- Domain joined
- Client certificate
- Specific software installed
</checks>

<example_policy>
"Allow access only if:
- Device is running macOS 13+
- FileVault is enabled
- WARP client is connected"
</example_policy>
</device_posture>

<common_patterns>

<pattern name="protect_admin_dashboard">
**Protect admin dashboard with SSO:**
1. Create Access application for admin.example.com
2. Configure Google/GitHub as IdP
3. Policy: Allow if email ends with @company.com
4. Set up Tunnel from origin to Cloudflare
</pattern>

<pattern name="secure_development_env">
**Secure staging environment:**
1. Tunnel connects staging server
2. Access requires authentication
3. Only team members can access
4. No public IP exposure
</pattern>

<pattern name="replace_vpn">
**Replace VPN for internal tools:**
1. Deploy Tunnel to internal network
2. Expose internal apps (Jira, Wiki, etc.)
3. Access policies per application
4. Users access via browser, no VPN client
</pattern>

</common_patterns>
