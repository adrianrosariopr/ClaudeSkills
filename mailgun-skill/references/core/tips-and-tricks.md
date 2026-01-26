# Mailgun Tips and Tricks

## API Shortcuts

### Quick Test Email via curl
Test sending without writing code:

```bash
# Minimal test
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/YOUR_DOMAIN/messages \
  -F from='test@YOUR_DOMAIN' \
  -F to='your@email.com' \
  -F subject='Test' \
  -F text='Hello!'

# With HTML
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/YOUR_DOMAIN/messages \
  -F from='test@YOUR_DOMAIN' \
  -F to='your@email.com' \
  -F subject='Test' \
  -F html='<h1>Hello!</h1>'
```

### Check Domain Status Quickly
```bash
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/domains/YOUR_DOMAIN | jq '.domain.state'
```

### List Recent Events
```bash
# Last 10 events
curl -s --user 'api:YOUR_API_KEY' \
  "https://api.mailgun.net/v3/YOUR_DOMAIN/events?limit=10" | jq '.items'

# Filter by event type
curl -s --user 'api:YOUR_API_KEY' \
  "https://api.mailgun.net/v3/YOUR_DOMAIN/events?event=bounced" | jq '.items'
```

### Search for Specific Email
```bash
# By recipient
curl -s --user 'api:YOUR_API_KEY' \
  "https://api.mailgun.net/v3/YOUR_DOMAIN/events?recipient=user@example.com"

# By message ID
curl -s --user 'api:YOUR_API_KEY' \
  "https://api.mailgun.net/v3/YOUR_DOMAIN/events?message-id=MESSAGE_ID"
```

---

## Dashboard Tricks

### Quick Domain Health Check
Dashboard → Sending → Domain Settings → DNS Records
- Green = properly configured
- Yellow = partial (check which record)
- Red = missing or incorrect

### Find Bounced Emails Fast
Dashboard → Sending → Logs → Filter by "Bounced"
- Export to CSV for bulk processing
- Click individual to see bounce reason

### Track Campaign Performance
Dashboard → Sending → Logs → Group by Tag
- Use `o:tag` when sending to group related emails
- Compare open/click rates across campaigns

### Set Up Alerts
Dashboard → Settings → Alerts
- Bounce rate threshold
- Complaint rate threshold
- Daily volume anomalies

---

## Laravel-Specific Tips

### Test Email in Tinker
```bash
php artisan tinker
```

```php
// Quick test
Mail::raw('Test message', fn($m) => $m->to('your@email.com'));

// Test mailable
Mail::to('your@email.com')->send(new App\Mail\WelcomeEmail($user));

// Preview mailable in browser
return (new App\Mail\WelcomeEmail($user))->render();
```

### Preview Mailables in Browser
```php
// routes/web.php (local only!)
if (app()->environment('local')) {
    Route::get('/mail-preview/{mailable}', function ($mailable) {
        $class = "App\\Mail\\{$mailable}";
        $user = User::first();
        return new $class($user);
    });
}
```

### Log Emails in Development
```php
// .env
MAIL_MAILER=log

// Emails written to storage/logs/laravel.log
```

### Queue Failed Email Retry
```bash
# See failed jobs
php artisan queue:failed

# Retry specific job
php artisan queue:retry JOB_ID

# Retry all failed
php artisan queue:retry all
```

### Custom Mailgun Headers
```php
class MyMailable extends Mailable
{
    public function build()
    {
        return $this->view('emails.template')
            ->withSymfonyMessage(function ($message) {
                $message->getHeaders()
                    ->addTextHeader('X-Mailgun-Tag', 'important')
                    ->addTextHeader('X-Mailgun-Variables', json_encode([
                        'user_id' => $this->user->id
                    ]));
            });
    }
}
```

---

## Node.js-Specific Tips

### TypeScript Types
```typescript
import Mailgun from 'mailgun.js';
import formData from 'form-data';

interface EmailData {
  from: string;
  to: string | string[];
  subject: string;
  text?: string;
  html?: string;
  template?: string;
  'h:X-Mailgun-Variables'?: string;
  'o:tag'?: string[];
}

const sendEmail = async (data: EmailData) => {
  return client.messages.create(domain, data);
};
```

### Reusable Client Setup
```javascript
// lib/mailgun.js
const Mailgun = require('mailgun.js');
const formData = require('form-data');

const mailgun = new Mailgun(formData);
const client = mailgun.client({
  username: 'api',
  key: process.env.MAILGUN_API_KEY,
  url: process.env.MAILGUN_REGION === 'eu'
    ? 'https://api.eu.mailgun.net'
    : 'https://api.mailgun.net'
});

module.exports = { client, domain: process.env.MAILGUN_DOMAIN };
```

### Async/Await Error Handling
```javascript
const sendEmail = async (data) => {
  try {
    const result = await client.messages.create(domain, data);
    console.log('Sent:', result.id);
    return { success: true, id: result.id };
  } catch (error) {
    console.error('Failed:', error.message);
    return { success: false, error: error.message };
  }
};
```

---

## Debugging Tips

### Check If Email Was Accepted
Mailgun returns a message ID immediately. This only means accepted, not delivered.

```javascript
const result = await client.messages.create(domain, data);
console.log('Message ID:', result.id);
// Use this ID to track in dashboard or events API
```

### Trace Email Journey
1. Check logs: Dashboard → Sending → Logs → Search by recipient
2. Check events: Accepted → Delivered (or Failed/Bounced)
3. Check suppressions: Dashboard → Suppressions

### Common Status Meanings
| Status | Meaning | Action |
|--------|---------|--------|
| Accepted | Mailgun received it | Wait for delivery |
| Delivered | Recipient server accepted | Success |
| Opened | Recipient opened (if tracking on) | Engagement |
| Clicked | Recipient clicked link | High engagement |
| Bounced | Delivery failed permanently | Remove address |
| Dropped | Suppressed (prior bounce/complaint) | Already removed |
| Complained | Marked as spam | Remove immediately |

### Debug Webhook Not Firing
1. Check webhook is registered: Dashboard → Sending → Webhooks
2. Check URL is accessible (not localhost)
3. Check for HTTPS requirement
4. Test with ngrok for local development:
   ```bash
   ngrok http 3000
   # Use the https URL in Mailgun webhook settings
   ```

---

## Performance Tips

### Batch Recipients Efficiently
Maximum 1000 recipients per API call:

```javascript
const chunkArray = (arr, size) =>
  Array.from({ length: Math.ceil(arr.length / size) },
    (_, i) => arr.slice(i * size, i * size + size));

const batches = chunkArray(recipients, 1000);
for (const batch of batches) {
  await client.messages.create(domain, {
    to: batch.map(r => r.email),
    // ...
  });
}
```

### Use Tags for Analytics
```javascript
await client.messages.create(domain, {
  // ...
  'o:tag': ['marketing', 'newsletter', '2024-01']
});
```
Then filter by tag in dashboard/API.

### Async Sending Pattern
```javascript
// Send multiple emails concurrently (with limit)
const pLimit = require('p-limit');
const limit = pLimit(10); // Max 10 concurrent

const results = await Promise.all(
  emails.map(email => limit(() => sendEmail(email)))
);
```

---

## Cost Optimization

### Use Templates Instead of HTML in API
Templates are stored in Mailgun, reducing payload size and API latency.

### Monitor Included Events
Free tier includes limited log retention. Export important events if needed.

### Clean Suppression Lists
Sending to suppressed addresses still counts against quota in some cases.

```bash
# List suppressions
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/YOUR_DOMAIN/bounces | jq '.items | length'
```

### Use Sandbox for Development
Sandbox domain is free and doesn't count against sending limits (except for authorized recipients).

---

## Security Tips

### Webhook Signature Verification
Always verify. Never skip.

```javascript
const crypto = require('crypto');

const verify = (signingKey, timestamp, token, signature) => {
  const encoded = crypto
    .createHmac('sha256', signingKey)
    .update(timestamp + token)
    .digest('hex');
  return encoded === signature;
};
```

### Rotate API Keys
Dashboard → Settings → API Security → API Keys
1. Create new key
2. Update your apps
3. Verify working
4. Delete old key

### Restrict SMTP Credentials
Create per-application SMTP credentials instead of using your main API key.

Dashboard → Sending → Domain Settings → SMTP Credentials

### Environment Variable Names
Standardize across projects:
```
MAILGUN_API_KEY=key-xxx
MAILGUN_DOMAIN=mail.yourapp.com
MAILGUN_REGION=us
MAILGUN_WEBHOOK_SIGNING_KEY=xxx
```
