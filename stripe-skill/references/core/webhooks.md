<overview>
Webhooks are HTTP callbacks from Stripe to your application. They notify you of events like payments, subscription changes, and more. Webhooks are the authoritative source for payment status - never rely solely on redirect URLs.
</overview>

<why_webhooks_matter>
**Problem with redirect URLs:**
```
User completes checkout → Redirect to success page
                          ↓
                          User closes browser
                          ↓
                          Your app never fulfills order
```

**Solution with webhooks:**
```
User completes checkout → Stripe sends webhook
                          ↓
                          Your app receives event
                          ↓
                          Order fulfilled (regardless of user behavior)
```

**Webhooks guarantee:**
- Event delivery (with retries)
- Correct event sequence
- Reliable payment confirmation
</why_webhooks_matter>

<essential_events>
**Must handle:**
| Event | Trigger | Action |
|-------|---------|--------|
| `customer.subscription.created` | New subscription | Grant access |
| `customer.subscription.updated` | Status/plan change | Update access |
| `customer.subscription.deleted` | Subscription ended | Revoke access |
| `invoice.payment_succeeded` | Payment successful | Log, send receipt |
| `invoice.payment_failed` | Payment failed | Notify user |

**Recommended:**
| Event | Trigger | Action |
|-------|---------|--------|
| `customer.updated` | Customer info changed | Sync data |
| `invoice.upcoming` | Invoice due soon | Notify user |
| `charge.succeeded` | Charge completed | Log |
| `charge.refunded` | Refund issued | Update records |
| `checkout.session.completed` | Checkout done | Fulfill one-time orders |
</essential_events>

<webhook_endpoint_setup>
**Stripe Dashboard:**
1. Developers → Webhooks → Add endpoint
2. Enter URL: `https://yourdomain.com/stripe/webhook`
3. Select events
4. Copy signing secret

**Laravel Cashier command:**
```bash
php artisan cashier:webhook --disabled
# Creates webhook in Stripe, outputs signing secret
```

**Local development:**
```bash
stripe listen --forward-to localhost:8000/stripe/webhook
# Outputs webhook signing secret for local testing
```
</webhook_endpoint_setup>

<signature_verification>
**Why verify signatures?**
- Prevents webhook spoofing
- Confirms event came from Stripe
- Required for security

**How it works:**
1. Stripe signs webhook payload with your secret
2. Your app verifies signature before processing
3. Invalid signatures are rejected (400 response)

**Laravel Cashier handles this automatically.**

**Manual verification (if needed):**
```php
$payload = @file_get_contents('php://input');
$sigHeader = $_SERVER['HTTP_STRIPE_SIGNATURE'];
$secret = config('cashier.webhook.secret');

try {
    $event = \Stripe\Webhook::constructEvent(
        $payload,
        $sigHeader,
        $secret
    );
} catch (\Stripe\Exception\SignatureVerificationException $e) {
    return response('Invalid signature', 400);
}

// Process $event
```
</signature_verification>

<handling_webhooks>
**Laravel Cashier built-in handling:**

Cashier automatically handles:
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`

**Extend for custom handling:**

```php
<?php

namespace App\Http\Controllers;

use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

class WebhookController extends CashierController
{
    protected function handleCustomerSubscriptionCreated(array $payload): void
    {
        parent::handleCustomerSubscriptionCreated($payload);

        // Custom logic
        $subscription = $payload['data']['object'];
        // Send welcome email, update permissions, etc.
    }

    protected function handleInvoicePaymentFailed(array $payload): void
    {
        parent::handleInvoicePaymentFailed($payload);

        // Custom logic
        $invoice = $payload['data']['object'];
        // Notify user, start dunning flow, etc.
    }
}
```

**Register custom controller:**
```php
// routes/web.php
Route::post('/stripe/webhook', [WebhookController::class, 'handleWebhook'])
    ->name('cashier.webhook');
```
</handling_webhooks>

<idempotency>
**Problem:** Stripe may send the same event multiple times.

**Solution:** Make handlers idempotent (same result regardless of how many times run).

```php
protected function handleInvoicePaymentSucceeded(array $payload): void
{
    $invoice = $payload['data']['object'];

    // Check if already processed
    if (Payment::where('stripe_invoice_id', $invoice['id'])->exists()) {
        Log::info('Already processed, skipping', ['id' => $invoice['id']]);
        return;
    }

    // Process (only runs once)
    Payment::create([
        'stripe_invoice_id' => $invoice['id'],
        // ...
    ]);

    parent::handleInvoicePaymentSucceeded($payload);
}
```
</idempotency>

<event_structure>
```json
{
    "id": "evt_xxxxx",
    "object": "event",
    "type": "customer.subscription.created",
    "created": 1234567890,
    "data": {
        "object": {
            "id": "sub_xxxxx",
            "customer": "cus_xxxxx",
            "status": "active",
            "items": {...},
            "current_period_start": 1234567890,
            "current_period_end": 1234567890
        },
        "previous_attributes": {}
    },
    "livemode": false,
    "pending_webhooks": 1,
    "request": {
        "id": "req_xxxxx",
        "idempotency_key": null
    }
}
```

**Access data:**
```php
$event = $payload;
$eventType = $event['type'];
$data = $event['data']['object'];
$previousAttributes = $event['data']['previous_attributes'] ?? [];
```
</event_structure>

<webhook_responses>
| Response | Meaning | Stripe Action |
|----------|---------|---------------|
| 2xx | Success | Event marked delivered |
| 3xx | Redirect | Follows redirect, then retries |
| 4xx | Client error | Retries (up to 3 days) |
| 5xx | Server error | Retries with exponential backoff |

**Return 200 quickly:**
```php
public function handleWebhook(Request $request)
{
    // Quick validation
    // ...

    // Queue heavy processing
    dispatch(new ProcessWebhook($payload));

    return response('Webhook received', 200);
}
```
</webhook_responses>

<retry_behavior>
Stripe retries failed webhooks:
- Immediate retry
- 1 hour
- 3 hours
- 24 hours
- 48 hours
- 72 hours (max)

**Monitor in Dashboard:**
Developers → Webhooks → [endpoint] → Recent events
</retry_behavior>

<testing_webhooks>
**Stripe CLI:**
```bash
# Forward to local
stripe listen --forward-to localhost:8000/stripe/webhook

# Trigger events
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
stripe trigger invoice.payment_failed
```

**Watch Laravel logs:**
```bash
tail -f storage/logs/laravel.log
```

**Dashboard:**
- Developers → Webhooks → [endpoint]
- Send test webhook
- View recent events and responses
</testing_webhooks>

<common_issues>
| Issue | Cause | Fix |
|-------|-------|-----|
| Signature verification failed | Wrong secret | Use correct `whsec_` |
| 404 on webhook URL | Route not registered | Check routes |
| 419 CSRF error | CSRF protection | Exclude `/stripe/*` |
| Events not processing | Handler not called | Check method naming |
| Duplicate processing | No idempotency | Add idempotency checks |
</common_issues>

<csrf_exclusion>
Webhooks can't include CSRF tokens. Exclude the route:

**Laravel 11+ (bootstrap/app.php):**
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

**Laravel 10 (VerifyCsrfToken.php):**
```php
protected $except = [
    'stripe/*',
];
```
</csrf_exclusion>
