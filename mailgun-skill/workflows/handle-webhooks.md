# Workflow: Handle Webhooks

<required_reading>
**Read these reference files NOW:**
1. references/webhooks-events.md
2. references/suppressions.md
3. references/nodejs-sdk.md (if Node.js)
4. references/laravel-driver.md (if Laravel)
</required_reading>

<process>
## Step 1: Understand Webhook Events

Mailgun sends POST requests when email events occur:

| Event | Triggered When |
|-------|----------------|
| `accepted` | Mailgun accepted the message for delivery |
| `delivered` | Message delivered to recipient server |
| `opened` | Recipient opened the email (requires tracking) |
| `clicked` | Recipient clicked a link (requires tracking) |
| `unsubscribed` | Recipient unsubscribed |
| `complained` | Recipient marked as spam |
| `temporary_fail` | Soft bounce, Mailgun will retry |
| `permanent_fail` | Hard bounce, Mailgun stopped trying |

**Critical events to handle:** `permanent_fail`, `complained`, `unsubscribed`

## Step 2: Configure Webhooks in Mailgun

1. Go to **Sending → Webhooks**
2. Click **Add Webhook**
3. Select event type (e.g., "Permanent Failure")
4. Enter your webhook URL (e.g., `https://yourapp.com/webhooks/mailgun`)
5. Repeat for each event type you want to track

You can add up to 3 URLs per event type.

## Step 3: Create Webhook Endpoint

### Node.js (Express)

```javascript
const express = require('express');
const crypto = require('crypto');
const router = express.Router();

// Verify webhook signature
function verifyWebhook(timestamp, token, signature) {
  const encodedToken = crypto
    .createHmac('sha256', process.env.MAILGUN_WEBHOOK_SIGNING_KEY)
    .update(timestamp.concat(token))
    .digest('hex');
  return encodedToken === signature;
}

router.post('/webhooks/mailgun', express.json(), async (req, res) => {
  const { signature, 'event-data': eventData } = req.body;

  // Verify signature
  if (!verifyWebhook(signature.timestamp, signature.token, signature.signature)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = eventData.event;
  const recipient = eventData.recipient;

  console.log(`Mailgun event: ${event} for ${recipient}`);

  // Handle different events
  switch (event) {
    case 'failed':
      if (eventData.severity === 'permanent') {
        await handlePermanentFailure(recipient, eventData);
      }
      break;

    case 'complained':
      await handleComplaint(recipient, eventData);
      break;

    case 'unsubscribed':
      await handleUnsubscribe(recipient, eventData);
      break;

    case 'delivered':
      await handleDelivery(recipient, eventData);
      break;

    case 'opened':
      await handleOpen(recipient, eventData);
      break;

    case 'clicked':
      await handleClick(recipient, eventData);
      break;
  }

  res.status(200).json({ received: true });
});

async function handlePermanentFailure(email, data) {
  // Remove from mailing list, mark as bounced
  console.log(`Hard bounce: ${email}`, data['delivery-status']);
  await db.users.update(
    { email },
    { emailStatus: 'bounced', bounceReason: data['delivery-status']?.message }
  );
}

async function handleComplaint(email, data) {
  // IMMEDIATELY stop sending to this address
  console.log(`Spam complaint: ${email}`);
  await db.users.update({ email }, { emailStatus: 'complained', subscribed: false });
}

async function handleUnsubscribe(email, data) {
  console.log(`Unsubscribed: ${email}`);
  await db.users.update({ email }, { subscribed: false });
}

async function handleDelivery(email, data) {
  console.log(`Delivered to: ${email}`);
  // Optional: track delivery metrics
}

async function handleOpen(email, data) {
  console.log(`Opened by: ${email}`);
  // Optional: track engagement
}

async function handleClick(email, data) {
  const url = data.url;
  console.log(`Clicked by: ${email}, URL: ${url}`);
  // Optional: track engagement
}

module.exports = router;
```

### Laravel

Create controller:
```bash
php artisan make:controller MailgunWebhookController
```

```php
// app/Http/Controllers/MailgunWebhookController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use App\Models\User;

class MailgunWebhookController extends Controller
{
    public function handle(Request $request)
    {
        // Verify signature
        if (!$this->verifySignature($request)) {
            return response()->json(['error' => 'Invalid signature'], 401);
        }

        $eventData = $request->input('event-data');
        $event = $eventData['event'];
        $recipient = $eventData['recipient'] ?? null;

        Log::info("Mailgun webhook: {$event}", ['recipient' => $recipient]);

        match ($event) {
            'failed' => $this->handleFailure($eventData),
            'complained' => $this->handleComplaint($recipient),
            'unsubscribed' => $this->handleUnsubscribe($recipient),
            'delivered' => $this->handleDelivery($recipient),
            'opened' => $this->handleOpen($recipient),
            'clicked' => $this->handleClick($eventData),
            default => null,
        };

        return response()->json(['received' => true]);
    }

    private function verifySignature(Request $request): bool
    {
        $signature = $request->input('signature');
        $timestamp = $signature['timestamp'] ?? '';
        $token = $signature['token'] ?? '';
        $providedSignature = $signature['signature'] ?? '';

        $expectedSignature = hash_hmac(
            'sha256',
            $timestamp . $token,
            config('services.mailgun.webhook_signing_key')
        );

        return hash_equals($expectedSignature, $providedSignature);
    }

    private function handleFailure(array $eventData): void
    {
        if (($eventData['severity'] ?? '') !== 'permanent') {
            return; // Ignore temporary failures
        }

        $email = $eventData['recipient'];
        $reason = $eventData['delivery-status']['message'] ?? 'Unknown';

        User::where('email', $email)->update([
            'email_status' => 'bounced',
            'bounce_reason' => $reason,
        ]);

        Log::warning("Hard bounce: {$email}", ['reason' => $reason]);
    }

    private function handleComplaint(string $email): void
    {
        User::where('email', $email)->update([
            'email_status' => 'complained',
            'subscribed' => false,
        ]);

        Log::warning("Spam complaint: {$email}");
    }

    private function handleUnsubscribe(string $email): void
    {
        User::where('email', $email)->update(['subscribed' => false]);
        Log::info("Unsubscribed: {$email}");
    }

    private function handleDelivery(string $email): void
    {
        Log::info("Delivered: {$email}");
    }

    private function handleOpen(string $email): void
    {
        Log::info("Opened: {$email}");
    }

    private function handleClick(array $eventData): void
    {
        $email = $eventData['recipient'];
        $url = $eventData['url'] ?? '';
        Log::info("Clicked: {$email}", ['url' => $url]);
    }
}
```

Add route (exclude from CSRF):
```php
// routes/web.php
Route::post('/webhooks/mailgun', [MailgunWebhookController::class, 'handle'])
    ->withoutMiddleware([\App\Http\Middleware\VerifyCsrfToken::class]);
```

Add webhook signing key to config:
```php
// config/services.php
'mailgun' => [
    // ... existing config
    'webhook_signing_key' => env('MAILGUN_WEBHOOK_SIGNING_KEY'),
],
```

## Step 4: Get Webhook Signing Key

1. Go to **Sending → Webhooks**
2. Click **Webhook signing key** to reveal
3. Add to `.env`:
```env
MAILGUN_WEBHOOK_SIGNING_KEY=your-signing-key-here
```

## Step 5: Test Webhooks Locally

Use ngrok or similar for local testing:

```bash
ngrok http 3000  # or 8000 for Laravel
```

Update webhook URL in Mailgun to your ngrok URL.

Or use Mailgun's test webhook feature in the dashboard.

## Step 6: Handle at Scale

For high-volume applications, process webhooks asynchronously:

### Node.js with Queue

```javascript
const Queue = require('bull');
const webhookQueue = new Queue('mailgun-webhooks');

// Endpoint just queues the job
router.post('/webhooks/mailgun', (req, res) => {
  webhookQueue.add(req.body);
  res.status(200).json({ queued: true });
});

// Process jobs asynchronously
webhookQueue.process(async (job) => {
  const { 'event-data': eventData } = job.data;
  // Handle event...
});
```

### Laravel with Queue

```php
// In controller
WebhookJob::dispatch($eventData);

// app/Jobs/WebhookJob.php
class WebhookJob implements ShouldQueue
{
    public function __construct(public array $eventData) {}

    public function handle(): void
    {
        // Process event...
    }
}
```
</process>

<success_criteria>
Webhook handling is complete when:
- [ ] Webhook endpoint created and accessible
- [ ] Signature verification working
- [ ] All critical events handled (permanent_fail, complained, unsubscribed)
- [ ] Database updated when bounces/complaints occur
- [ ] Logs show webhook activity
- [ ] Tested with real webhook events
</success_criteria>

<anti_patterns>
Avoid:
- Skipping signature verification (security risk)
- Not handling complaints (can damage sender reputation)
- Blocking request while processing (use queues for heavy work)
- Continuing to send to bounced/complained addresses
- Returning non-200 response (Mailgun will retry)
- Forgetting to exclude webhook route from CSRF (Laravel)
</anti_patterns>
