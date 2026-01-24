<overview>
Mailgun has various rate limits to protect service quality. Understanding and handling these limits ensures reliable email delivery.
</overview>

<account_limits>
## Account Sending Limits

<free_tier>
**Free Tier:**
- 100 emails per day
- Sandbox domain only sends to verified recipients
- May have hourly limits while unverified
</free_tier>

<paid_plans>
**Paid Plans:**
- Limits based on plan tier
- Typically 10,000-100,000+ per month
- Higher tiers have higher burst rates
</paid_plans>

<new_accounts>
**New Accounts:**
- May have temporary hourly limits
- Complete business verification to remove
- Dashboard: Account → Settings → Business Verification
</new_accounts>
</account_limits>

<api_rate_limits>
## API Rate Limits

<general_limits>
**General API:**
- Rate varies by endpoint and plan
- Check `X-RateLimit-*` response headers
- 429 status when exceeded
</general_limits>

<response_headers>
**Rate limit headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1234567890
```
</response_headers>

<validation_api>
**Validation API:**
- Limited concurrent requests
- Returns 429 when at capacity
- Implement queuing for bulk validation
</validation_api>
</api_rate_limits>

<esp_throttling>
## ESP/Provider Throttling

<mailbox_provider_limits>
**Mailbox providers have their own limits:**

| Provider | Behavior |
|----------|----------|
| Gmail | Throttles new senders, requires warm-up |
| Microsoft | Strict reputation-based limits |
| Yahoo | Connection limits per IP |
| AOL | Similar to Yahoo (same company) |

Mailgun automatically adjusts sending rate based on provider feedback.
</mailbox_provider_limits>

<reputation_throttling>
**Reputation-based throttling:**
- Poor reputation = slower delivery
- Too many bounces = temporary blocks
- Complaints trigger additional scrutiny
</reputation_throttling>
</esp_throttling>

<handling_limits>
## Handling Rate Limits

<exponential_backoff>
**Exponential backoff:**
```javascript
async function sendWithBackoff(data, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await mg.messages.create(domain, data);
    } catch (error) {
      if (error.status === 429) {
        // Calculate delay: 1s, 2s, 4s, 8s, 16s
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Rate limited, waiting ${delay}ms`);
        await new Promise(r => setTimeout(r, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```
</exponential_backoff>

<queue_based>
**Queue-based approach:**
```javascript
const Queue = require('bull');
const emailQueue = new Queue('emails');

// Add to queue instead of sending directly
emailQueue.add({ to, subject, html }, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000
  }
});

// Process with rate limiting
emailQueue.process(async (job) => {
  return mg.messages.create(domain, job.data);
}, { limiter: { max: 100, duration: 1000 } }); // 100 per second
```
</queue_based>

<laravel_throttle>
**Laravel with throttling:**
```php
// Using Laravel's rate limiter
RateLimiter::for('mailgun', function (Request $request) {
    return Limit::perMinute(100);
});

// Or with queue rate limiting
// config/queue.php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        'block_for' => 5,
        'retry_after' => 90,
    ],
],

// In job
public $tries = 3;
public $backoff = [60, 120, 300];
```
</laravel_throttle>
</handling_limits>

<batch_sending>
## Batch Sending Strategies

<recipient_batching>
**Batch recipients:**
Send to multiple recipients in one API call:

```javascript
// Instead of 100 API calls
for (const user of users) {
  await mg.messages.create(domain, { to: user.email, ... });
}

// Make 1 API call with recipient variables
await mg.messages.create(domain, {
  to: users.map(u => u.email),
  subject: 'Hello %recipient.name%',
  text: 'Hi %recipient.name%',
  'recipient-variables': JSON.stringify(
    Object.fromEntries(users.map(u => [u.email, { name: u.name }]))
  )
});
```

Max 1,000 recipients per batch.
</recipient_batching>

<chunking>
**Chunking large lists:**
```javascript
function chunk(array, size) {
  const chunks = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

const userChunks = chunk(users, 1000);

for (const batch of userChunks) {
  await mg.messages.create(domain, {
    to: batch.map(u => u.email),
    // ... recipient variables
  });

  // Optional: delay between batches
  await new Promise(r => setTimeout(r, 1000));
}
```
</chunking>
</batch_sending>

<monitoring_usage>
## Monitoring Usage

<dashboard>
**Via Dashboard:**
- Account → Usage → View sending statistics
- See daily/monthly totals
- Monitor approaching limits
</dashboard>

<api>
**Via API:**
```javascript
// Get domain stats
const stats = await mg.stats.getDomain(domain, {
  event: ['accepted', 'delivered', 'failed'],
  start: new Date(Date.now() - 86400000).toISOString(), // Last 24h
  resolution: 'day'
});

console.log('Sent:', stats.stats[0].accepted.total);
```
</api>
</monitoring_usage>

<best_practices>
## Best Practices

<do>
**Do:**
- Implement exponential backoff
- Use queues for high volume
- Batch recipients when possible
- Monitor rate limit headers
- Warm up gradually
</do>

<dont>
**Don't:**
- Ignore 429 responses
- Send maximum volume immediately
- Retry without delay
- Exceed recommended sending rates
</dont>
</best_practices>
