<overview>
Mailgun API keys authenticate your requests. Proper key management is essential for security.
</overview>

<key_types>
## API Key Types

<private_key>
**Private API Key**
- Full access to all API endpoints
- Used for server-side operations
- **Never expose in client-side code**
- Format: `key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
</private_key>

<public_key>
**Public Validation Key**
- Only for email validation API
- Safe to use in client-side code
- Format: `pubkey-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
</public_key>

<webhook_signing_key>
**Webhook Signing Key**
- Verifies webhook authenticity
- Used to validate incoming webhooks
- Different from API key
</webhook_signing_key>
</key_types>

<obtaining_keys>
## Obtaining Keys

1. Log in to Mailgun Dashboard
2. Go to **Account Settings → API Security**
3. View/copy your keys:
   - Private API key (click to reveal)
   - Public validation key
   - Webhook signing key (in Webhooks section)
</obtaining_keys>

<storage_best_practices>
## Storage Best Practices

<environment_variables>
**Use environment variables:**

```bash
# .env (never commit this file)
MAILGUN_API_KEY=key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MAILGUN_DOMAIN=mail.yourdomain.com
MAILGUN_REGION=us
MAILGUN_WEBHOOK_SIGNING_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Access in code:**
```javascript
// Node.js
const apiKey = process.env.MAILGUN_API_KEY;
```

```php
// Laravel
$apiKey = config('services.mailgun.secret');
```
</environment_variables>

<gitignore>
**Always gitignore sensitive files:**
```
# .gitignore
.env
.env.local
.env.*.local
*.pem
```
</gitignore>

<secrets_management>
**Production secrets management:**
- AWS Secrets Manager
- HashiCorp Vault
- Google Cloud Secret Manager
- Azure Key Vault
- Vercel/Netlify environment variables
- Railway/Render secrets
</secrets_management>
</storage_best_practices>

<security_practices>
## Security Practices

<key_rotation>
**Key rotation:**
1. Generate new key in Mailgun dashboard
2. Update application with new key
3. Verify application works
4. Delete old key

Rotate keys:
- Immediately if compromised
- Periodically (every 6-12 months)
- When team members leave
</key_rotation>

<access_control>
**Limit key access:**
- Only give keys to services that need them
- Use different keys for different environments
- Consider using domain-specific keys
</access_control>

<monitoring>
**Monitor for misuse:**
- Set up usage alerts in Mailgun
- Monitor for unexpected sending patterns
- Review API logs regularly
</monitoring>
</security_practices>

<leaked_key_response>
## If Key is Leaked

**Immediate actions:**
1. Go to Mailgun Dashboard → API Security
2. Generate new API key
3. Delete the compromised key
4. Update all applications with new key
5. Review logs for unauthorized activity
6. Check for unexpected suppressions or changes

**Prevention:**
- Use git-secrets or similar tools
- Scan repositories for leaked secrets
- Use .env files, never hardcode
</leaked_key_response>

<per_environment>
## Environment-Specific Keys

<development>
**Development:**
- Use sandbox domain
- Separate API key if possible
- Limited sending (test recipients only)
</development>

<staging>
**Staging:**
- Use test domain (if available)
- May share with production key
- Send to test addresses only
</staging>

<production>
**Production:**
- Verified domain with DNS
- Production API key
- Full sending capability
</production>

**Example .env structure:**
```bash
# .env.development
MAILGUN_API_KEY=key-dev-xxxxx
MAILGUN_DOMAIN=sandbox123.mailgun.org

# .env.production
MAILGUN_API_KEY=key-prod-xxxxx
MAILGUN_DOMAIN=mail.yourdomain.com
```
</per_environment>
