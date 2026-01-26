<overview>
Common issues encountered when integrating Mailgun, with solutions and troubleshooting steps.
</overview>

<authentication_issues>
## Authentication Issues

<invalid_api_key>
**401 Unauthorized / Invalid API Key**

**Symptoms:**
- API returns 401 status
- "Invalid API key" error message

**Causes:**
1. Wrong API key (copied incorrectly)
2. Wrong region (US key with EU endpoint or vice versa)
3. API key revoked or regenerated

**Solutions:**
```javascript
// Verify key format (should start with "key-")
console.log(process.env.MAILGUN_API_KEY?.substring(0, 4)); // "key-"

// Check region matches key
const url = process.env.MAILGUN_REGION === 'eu'
  ? 'https://api.eu.mailgun.net'
  : 'https://api.mailgun.net';
```

**Verification:**
1. Go to Mailgun Dashboard → Account → API Security
2. Copy the Private API key (not Public key)
3. Note which region the key belongs to
</invalid_api_key>

<domain_not_found>
**404 Domain Not Found**

**Symptoms:**
- API returns 404 status
- "Domain not found" error

**Causes:**
1. Domain not added to Mailgun
2. Typo in domain name
3. Using wrong region

**Solutions:**
```javascript
// Check domain is correct
console.log(process.env.MAILGUN_DOMAIN); // "mail.yourdomain.com"

// Verify domain exists in Mailgun dashboard
```
</domain_not_found>
</authentication_issues>

<delivery_issues>
## Delivery Issues

<emails_going_to_spam>
**Emails Landing in Spam**

**Common causes:**
1. DNS not configured (SPF/DKIM missing)
2. Using sandbox domain in production
3. Poor sender reputation
4. Spam trigger content

**Diagnosis:**
1. Check DNS: Dashboard → Domains → Verify DNS Settings
2. Check sender score: https://senderscore.org
3. Test content: https://mail-tester.com

**Solutions:**
- Complete DNS setup (see dns-configuration.md)
- Use a subdomain, not sandbox
- Remove spam trigger words
- Add unsubscribe link
- Warm up new domains gradually
</emails_going_to_spam>

<accepted_not_delivered>
**Emails Accepted but Not Delivered**

**Symptoms:**
- Mailgun shows "Accepted"
- No "Delivered" event
- Recipient never receives email

**Common causes:**
1. Recipient server MX issues
2. Email scheduled for later
3. Still in queue (check back later)
4. Greylisting (first-time sender delay)

**Diagnosis:**
```bash
# Check recipient's MX records
dig MX recipientdomain.com +short
```

**Solutions:**
- Wait for delivery (up to 8 hours for retries)
- Check if scheduled delivery was used
- Contact recipient to check spam folder
</accepted_not_delivered>

<too_old_status>
**"Too Old" Status**

**Symptoms:**
- Permanent failure with "Too old" message
- Message expired before delivery

**Causes:**
- Recipient server consistently unavailable
- Rate limiting from poor reputation
- Network issues

**Solutions:**
1. Check recipient server status
2. Review sender reputation
3. Try alternative email for that domain
</too_old_status>
</delivery_issues>

<sandbox_limitations>
## Sandbox Domain Limitations

<sandbox_restrictions>
**Free tier sandbox restrictions:**
- Can only send to verified recipients
- Limited to 5 authorized recipients
- 100 emails per day

**To add authorized recipients:**
1. Dashboard → Sending → Domains → Sandbox domain
2. Click "Authorized Recipients"
3. Add email addresses
4. Recipients must verify via confirmation email

**Solution:** Verify your own domain for production use.
</sandbox_restrictions>
</sandbox_limitations>

<rate_limiting>
## Rate Limiting

<429_errors>
**429 Too Many Requests**

**Symptoms:**
- API returns 429 status
- "Rate limit exceeded" message

**Causes:**
1. Sending too fast
2. Account hourly limit reached
3. Business verification pending

**Solutions:**
```javascript
// Implement exponential backoff
async function sendWithBackoff(data, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await mg.messages.create(domain, data);
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(r => setTimeout(r, delay));
      } else {
        throw error;
      }
    }
  }
}
```

**Check limits:**
- Dashboard → Account → Settings
- Complete business verification if prompted
</429_errors>
</rate_limiting>

<webhook_issues>
## Webhook Issues

<webhooks_not_received>
**Webhooks Not Being Received**

**Diagnosis:**
1. Is URL publicly accessible?
2. Does endpoint return 200?
3. Is SSL certificate valid?

**Common fixes:**
- Ensure HTTPS with valid certificate
- Return 200 status (not redirect)
- Check firewall allows Mailgun IPs
- Test with ngrok locally
</webhooks_not_received>

<signature_verification_failing>
**Signature Verification Failing**

**Causes:**
1. Wrong signing key
2. Request body modified by middleware
3. Encoding issues

**Solutions:**
```javascript
// Ensure raw body is preserved
app.use('/webhooks/mailgun', express.raw({ type: 'application/json' }));

// Or use correct middleware order
app.post('/webhooks/mailgun', express.json(), handler);
```

**Get correct signing key:**
Dashboard → Sending → Webhooks → "Webhook signing key"
</signature_verification_failing>

<csrf_laravel>
**Laravel CSRF Errors**

**Symptom:** 419 Page Expired

**Solution:** Exclude webhook route from CSRF:
```php
// app/Http/Middleware/VerifyCsrfToken.php
protected $except = [
    'webhooks/mailgun',
];

// Or in route definition
Route::post('/webhooks/mailgun', [WebhookController::class, 'handle'])
    ->withoutMiddleware([\App\Http\Middleware\VerifyCsrfToken::class]);
```
</csrf_laravel>
</webhook_issues>

<tracking_issues>
## Tracking Issues

<opens_not_tracking>
**Opens Not Being Tracked**

**Requirements:**
1. Tracking CNAME record configured
2. `o:tracking-opens: yes` in request
3. HTML content (tracking pixel needed)
4. Recipient email client loads images

**Check CNAME:**
```bash
dig CNAME email.mail.yourdomain.com +short
# Should return: mailgun.org
```

**Note:** Some email clients block tracking pixels by default.
</opens_not_tracking>

<clicks_not_tracking>
**Clicks Not Being Tracked**

**Requirements:**
1. Tracking CNAME configured
2. `o:tracking-clicks: yes` in request
3. Links in HTML content (not plain text)

**Verify links are trackable:**
- Must be full URLs (https://...)
- Not JavaScript links
- Not mailto: links
</clicks_not_tracking>
</tracking_issues>

<region_issues>
## Region Mismatch

<us_vs_eu>
**US vs EU Region**

**Symptoms:**
- 401 errors despite correct API key
- Domain not found errors

**The issue:**
- US accounts use `api.mailgun.net`
- EU accounts use `api.eu.mailgun.net`
- Keys and domains are region-specific

**Solution:**
```javascript
// Node.js
const url = process.env.MAILGUN_REGION === 'eu'
  ? 'https://api.eu.mailgun.net'
  : 'https://api.mailgun.net';
```

```php
// Laravel config/services.php
'mailgun' => [
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
],
```

**Check your region:**
Look at your Mailgun dashboard URL:
- `app.mailgun.com` = US
- `app.eu.mailgun.net` = EU
</us_vs_eu>
</region_issues>
