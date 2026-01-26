# Mailgun Best Practices

## Domain Configuration

### Use Dedicated Subdomains
```
Good:  mail.yourapp.com, notifications.yourapp.com
Bad:   yourapp.com (root domain)
```

Why: Protects main domain reputation, allows separate tracking, avoids MX record conflicts.

### Complete DNS Authentication
All three records are essential:
- **SPF** - Tells receivers Mailgun is authorized to send for you
- **DKIM** - Cryptographically signs emails to prove authenticity
- **DMARC** - Tells receivers what to do with failed authentication

```dns
# SPF Record (TXT)
v=spf1 include:mailgun.org ~all

# DKIM Record (TXT) - Mailgun provides this
k=rsa; p=MIGfMA0GCSqGSIb3DQEBA...

# DMARC Record (TXT)
v=DMARC1; p=quarantine; rua=mailto:dmarc@yourapp.com
```

### Verify Before Sending Production Email
Never send production email until domain verification shows all green checkmarks. Unverified domains have poor deliverability.

---

## Sending Patterns

### Queue All Email
Never send email synchronously in web requests:

```php
// Laravel - Good (queued)
Mail::to($user)->queue(new WelcomeEmail($user));

// Laravel - Bad (blocks request)
Mail::to($user)->send(new WelcomeEmail($user));
```

```javascript
// Node.js - Use job queue (Bull, BullMQ, etc.)
await emailQueue.add('send-welcome', { userId: user.id });

// Process in worker
emailQueue.process('send-welcome', async (job) => {
  await mailgun.messages.create(domain, emailData);
});
```

### Use Templates for Repeated Emails
Store templates in Mailgun for:
- Consistent branding
- Non-developer editing
- Faster sending (no HTML in API call)
- A/B testing support

```javascript
// Reference stored template
await mailgun.messages.create(domain, {
  to: user.email,
  template: 'welcome-email',
  'h:X-Mailgun-Variables': JSON.stringify({
    name: user.name,
    action_url: 'https://app.com/activate'
  })
});
```

### Batch Sending for Bulk Operations
Use recipient variables for bulk sends instead of individual API calls:

```javascript
// Good - Single API call for 1000 recipients
await mailgun.messages.create(domain, {
  to: recipients.map(r => r.email),
  subject: 'Newsletter',
  template: 'newsletter',
  'recipient-variables': JSON.stringify(
    recipients.reduce((acc, r) => ({
      ...acc,
      [r.email]: { name: r.name, id: r.id }
    }), {})
  )
});

// Bad - 1000 API calls
for (const recipient of recipients) {
  await mailgun.messages.create(domain, { ... });
}
```

---

## Webhook Handling

### Always Verify Webhook Signatures
Webhooks can be spoofed. Always verify the signature:

```javascript
const crypto = require('crypto');

function verifyWebhook(timestamp, token, signature, signingKey) {
  const encodedToken = crypto
    .createHmac('sha256', signingKey)
    .update(timestamp + token)
    .digest('hex');
  return encodedToken === signature;
}

// In webhook handler
const { timestamp, token, signature } = req.body.signature;
if (!verifyWebhook(timestamp, token, signature, process.env.MAILGUN_WEBHOOK_KEY)) {
  return res.status(401).send('Invalid signature');
}
```

### Process Webhooks Asynchronously
Don't do heavy processing in the webhook handler. Queue the work:

```javascript
// Good - Quick response, process later
app.post('/webhooks/mailgun', async (req, res) => {
  await webhookQueue.add('process-mailgun', req.body);
  res.status(200).send('OK'); // Respond immediately
});

// Bad - Mailgun may timeout
app.post('/webhooks/mailgun', async (req, res) => {
  await updateUserEmailStatus(req.body); // Slow
  await logToAnalytics(req.body); // Slow
  res.status(200).send('OK');
});
```

### Handle All Bounce Types
Different bounces require different actions:

| Event | Action |
|-------|--------|
| `bounced` (hard) | Remove from list permanently |
| `dropped` (prior bounce) | Already suppressed, log only |
| `complained` | Remove immediately, add to suppression |
| `unsubscribed` | Honor preference, update subscription |

---

## Deliverability

### Warm Up New Domains
Don't send thousands of emails from a new domain immediately:

| Day | Volume |
|-----|--------|
| 1-3 | 50-100 |
| 4-7 | 200-500 |
| 8-14 | 500-1000 |
| 15+ | Gradually increase |

### Maintain List Hygiene
- Remove hard bounces immediately
- Remove addresses with repeated soft bounces (3+)
- Remove complainers immediately
- Implement double opt-in for signups
- Re-engagement campaigns before removing inactive users

### Monitor Engagement Metrics
Track in Mailgun dashboard:
- **Delivery rate** - Should be >98%
- **Open rate** - Industry varies, 15-25% typical
- **Click rate** - 2-5% typical
- **Bounce rate** - Keep <2%
- **Complaint rate** - Keep <0.1%

### Personalize Content
Personalized emails have better engagement:

```javascript
await mailgun.messages.create(domain, {
  to: user.email,
  subject: `${user.name}, your weekly summary`,
  template: 'weekly-summary',
  'h:X-Mailgun-Variables': JSON.stringify({
    name: user.name,
    stats: user.weeklyStats,
    recommendations: user.recommendations
  })
});
```

---

## Security

### Rotate API Keys Periodically
- Generate new key
- Update applications
- Verify functionality
- Delete old key

### Use Domain-Specific Sending Keys
Create separate sending credentials for each application/environment:
- `app-production@mail.yourapp.com`
- `app-staging@mail.yourapp.com`
- `marketing@mail.yourapp.com`

### Restrict Key Permissions
Use scoped API keys when possible:
- Sending-only keys for application code
- Full-access keys only for admin operations

### Never Log Full API Keys
```javascript
// Good
console.log(`Using key: ${apiKey.slice(0, 8)}...`);

// Bad
console.log(`Using key: ${apiKey}`);
```

---

## Error Handling

### Implement Retry Logic
Transient failures should be retried:

```javascript
async function sendWithRetry(emailData, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await mailgun.messages.create(domain, emailData);
    } catch (error) {
      if (attempt === maxRetries || !isRetryable(error)) {
        throw error;
      }
      await sleep(Math.pow(2, attempt) * 1000); // Exponential backoff
    }
  }
}

function isRetryable(error) {
  const retryableCodes = [429, 500, 502, 503, 504];
  return retryableCodes.includes(error.status);
}
```

### Log Failures for Investigation
```javascript
try {
  const result = await mailgun.messages.create(domain, emailData);
  logger.info('Email sent', { messageId: result.id, to: emailData.to });
} catch (error) {
  logger.error('Email failed', {
    to: emailData.to,
    subject: emailData.subject,
    error: error.message,
    details: error.details
  });
  // Re-throw or handle appropriately
}
```

---

## Testing

### Use Mailgun Sandbox for Development
The sandbox domain allows testing without DNS setup:
```
sandbox123abc.mailgun.org
```

### Test with Real Addresses in Staging
Use your own addresses to verify:
- Email actually arrives
- Formatting looks correct
- Links work
- Unsubscribe works

### Verify Webhooks with Mailgun CLI
```bash
# Trigger test webhook events
curl -X POST https://api.mailgun.net/v3/domains/YOUR_DOMAIN/webhooks \
  --user 'api:YOUR_KEY' \
  -F url='https://yourapp.com/webhooks/mailgun' \
  -F event='clicked'
```
