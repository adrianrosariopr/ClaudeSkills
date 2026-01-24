<overview>
Mailgun webhooks send HTTP POST requests to your server when email events occur. This enables real-time tracking of deliveries, bounces, opens, clicks, and complaints.
</overview>

<event_types>
## Event Types

| Event | Description | When to Use |
|-------|-------------|-------------|
| `accepted` | Message accepted by Mailgun | Track submission success |
| `delivered` | Delivered to recipient server | Confirm delivery |
| `opened` | Recipient opened email | Track engagement |
| `clicked` | Recipient clicked link | Track engagement |
| `unsubscribed` | Recipient unsubscribed | Update preferences |
| `complained` | Marked as spam | Stop sending immediately |
| `temporary_fail` | Soft bounce, will retry | Monitor (usually auto-handled) |
| `permanent_fail` | Hard bounce, stopped | Remove from list |
| `stored` | Stored for retrieval | Inbound email processing |

<critical_events>
**Critical events to handle:**
- `permanent_fail` - Remove address from your list
- `complained` - Stop all email to this address
- `unsubscribed` - Honor unsubscribe request
</critical_events>
</event_types>

<webhook_payload>
## Webhook Payload Structure

```json
{
  "signature": {
    "timestamp": "1529006854",
    "token": "a8ce0edb2dd8301dee6c2405235584e45aa91d1e9f979f3de0",
    "signature": "d2271d12299f6592d9d44cd9d250f0704e4674c30d79d07c47a66f95ce71cf55"
  },
  "event-data": {
    "event": "delivered",
    "timestamp": 1529006854.329574,
    "id": "CPgfbmQMTCKtHW6uIWtuVe",
    "recipient": "user@example.com",
    "recipient-domain": "example.com",
    "message": {
      "headers": {
        "message-id": "20180614184944.1.0F7C3A8C0@example.com",
        "from": "sender@yourdomain.com",
        "to": "user@example.com",
        "subject": "Test Subject"
      },
      "size": 1234
    },
    "tags": ["welcome-email"],
    "user-variables": {
      "user-id": "12345"
    },
    "delivery-status": {
      "message": "OK",
      "code": 250,
      "description": ""
    }
  }
}
```
</webhook_payload>

<failed_event_payload>
## Failed Event Payload

```json
{
  "event-data": {
    "event": "failed",
    "severity": "permanent",
    "reason": "bounce",
    "timestamp": 1529006854.329574,
    "recipient": "invalid@example.com",
    "delivery-status": {
      "code": 550,
      "bounce-code": "5.1.1",
      "message": "The email account does not exist",
      "description": "Not delivering to previously bounced address",
      "enhanced-code": "5.1.1"
    },
    "flags": {
      "is-delayed-bounce": false
    }
  }
}
```

<severity_types>
**Severity:**
- `permanent` - Hard bounce, stop sending
- `temporary` - Soft bounce, Mailgun will retry
</severity_types>

<reason_codes>
**Reason codes:**
- `bounce` - Recipient server rejected
- `suppress-bounce` - On Mailgun's bounce suppression list
- `suppress-complaint` - Previously complained
- `suppress-unsubscribe` - Previously unsubscribed
- `generic` - Other permanent failure
</reason_codes>
</failed_event_payload>

<signature_verification>
## Signature Verification

**Always verify webhook signatures** to prevent spoofed requests.

<node_verification>
**Node.js:**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(signingKey, timestamp, token, signature) {
  const encodedToken = crypto
    .createHmac('sha256', signingKey)
    .update(timestamp + token)
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(encodedToken),
    Buffer.from(signature)
  );
}

// Usage
const { signature } = req.body;
const isValid = verifyWebhookSignature(
  process.env.MAILGUN_WEBHOOK_SIGNING_KEY,
  signature.timestamp,
  signature.token,
  signature.signature
);
```
</node_verification>

<php_verification>
**PHP/Laravel:**
```php
function verifyWebhookSignature(
    string $signingKey,
    string $timestamp,
    string $token,
    string $signature
): bool {
    $expectedSignature = hash_hmac('sha256', $timestamp . $token, $signingKey);
    return hash_equals($expectedSignature, $signature);
}
```
</php_verification>

<signing_key>
**Get signing key:**
1. Go to Mailgun Dashboard → Sending → Webhooks
2. Click "Webhook signing key" to reveal
3. Store in environment variable: `MAILGUN_WEBHOOK_SIGNING_KEY`
</signing_key>
</signature_verification>

<webhook_setup>
## Setting Up Webhooks

<via_dashboard>
**Via Dashboard:**
1. Go to Sending → Webhooks
2. Click "Add webhook"
3. Select event type
4. Enter your webhook URL
5. Repeat for each event type

You can add up to 3 URLs per event type.
</via_dashboard>

<via_api>
**Via API (Node.js):**
```javascript
await mg.webhooks.create(domain, 'permanent_fail', 'https://yourapp.com/webhooks/mailgun');
await mg.webhooks.create(domain, 'complained', 'https://yourapp.com/webhooks/mailgun');
await mg.webhooks.create(domain, 'delivered', 'https://yourapp.com/webhooks/mailgun');
```
</via_api>
</webhook_setup>

<tracking_events>
## Tracking Opens and Clicks

**Requirements:**
1. Tracking CNAME record configured
2. `o:tracking`, `o:tracking-opens`, `o:tracking-clicks` enabled
3. HTML content (opens require a tracking pixel)

<open_event>
**Open event payload:**
```json
{
  "event-data": {
    "event": "opened",
    "recipient": "user@example.com",
    "timestamp": 1529006854.329574,
    "geolocation": {
      "country": "US",
      "region": "CA",
      "city": "San Francisco"
    },
    "client-info": {
      "client-type": "browser",
      "client-os": "macOS",
      "device-type": "desktop",
      "client-name": "Chrome",
      "user-agent": "Mozilla/5.0..."
    },
    "ip": "50.56.129.169"
  }
}
```
</open_event>

<click_event>
**Click event payload:**
```json
{
  "event-data": {
    "event": "clicked",
    "recipient": "user@example.com",
    "url": "https://example.com/promo",
    "timestamp": 1529006854.329574,
    "geolocation": {...},
    "client-info": {...}
  }
}
```
</click_event>
</tracking_events>

<best_practices>
## Best Practices

<respond_quickly>
**Respond quickly:**
- Return 200 status immediately
- Process heavy work asynchronously (queue)
- Mailgun retries on non-2xx responses
</respond_quickly>

<idempotency>
**Handle duplicates:**
- Mailgun may send the same event multiple times
- Use `event-data.id` to deduplicate
- Make handlers idempotent
</idempotency>

<example_handler>
**Complete handler pattern:**
```javascript
const processedEvents = new Set(); // Or use Redis/DB

app.post('/webhooks/mailgun', async (req, res) => {
  // 1. Respond immediately
  res.status(200).json({ received: true });

  // 2. Verify signature
  if (!verifySignature(req.body.signature)) {
    console.error('Invalid webhook signature');
    return;
  }

  // 3. Deduplicate
  const eventId = req.body['event-data'].id;
  if (processedEvents.has(eventId)) {
    return;
  }
  processedEvents.add(eventId);

  // 4. Queue for async processing
  await webhookQueue.add(req.body);
});
```
</example_handler>
</best_practices>
