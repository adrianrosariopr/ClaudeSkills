# Workflow: Implement Webhooks

<required_reading>
**Read these reference files NOW:**
1. references/webhooks.md
2. references/security.md
</required_reading>

<process>

## Step 1: Understand Webhook Flow

```
User completes checkout on Stripe
         │
         ▼
Stripe processes payment
         │
         ├──▶ User redirected to success_url (unreliable)
         │
         ▼
Stripe sends webhook to your endpoint (reliable)
         │
         ▼
Your app verifies signature & processes event
         │
         ▼
Database updated with subscription/payment status
```

**Webhooks are the source of truth. Never rely solely on redirects.**

## Step 2: Configure Webhook Events in Stripe

In Stripe Dashboard → **Developers** → **Webhooks**:

**Required events:**
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`

**Recommended events:**
- `customer.updated`
- `invoice.upcoming`
- `charge.succeeded`
- `charge.failed`
- `charge.refunded`

## Step 3: Laravel Cashier's Built-in Webhook Handling

Cashier automatically handles common events. The webhook route is registered at `/stripe/webhook`.

To customize handling, extend the webhook controller.

Create `app/Http/Controllers/WebhookController.php`:

```php
<?php

namespace App\Http\Controllers;

use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;

class WebhookController extends CashierController
{
    /**
     * Handle customer subscription created.
     */
    protected function handleCustomerSubscriptionCreated(array $payload): void
    {
        parent::handleCustomerSubscriptionCreated($payload);

        $subscription = $payload['data']['object'];

        Log::info('Subscription created', [
            'subscription_id' => $subscription['id'],
            'customer' => $subscription['customer'],
            'status' => $subscription['status'],
        ]);

        // Custom logic: Send welcome email, update permissions, etc.
        // $user = $this->getUserByStripeId($subscription['customer']);
        // Mail::to($user)->send(new SubscriptionWelcome($subscription));
    }

    /**
     * Handle customer subscription updated.
     */
    protected function handleCustomerSubscriptionUpdated(array $payload): void
    {
        parent::handleCustomerSubscriptionUpdated($payload);

        $subscription = $payload['data']['object'];

        Log::info('Subscription updated', [
            'subscription_id' => $subscription['id'],
            'status' => $subscription['status'],
            'cancel_at_period_end' => $subscription['cancel_at_period_end'] ?? false,
        ]);

        // Handle plan changes, cancellations, etc.
    }

    /**
     * Handle customer subscription deleted.
     */
    protected function handleCustomerSubscriptionDeleted(array $payload): void
    {
        parent::handleCustomerSubscriptionDeleted($payload);

        $subscription = $payload['data']['object'];

        Log::info('Subscription deleted', [
            'subscription_id' => $subscription['id'],
            'customer' => $subscription['customer'],
        ]);

        // Revoke access, send cancellation confirmation, etc.
    }

    /**
     * Handle invoice payment succeeded.
     */
    protected function handleInvoicePaymentSucceeded(array $payload): void
    {
        parent::handleInvoicePaymentSucceeded($payload);

        $invoice = $payload['data']['object'];

        Log::info('Invoice paid', [
            'invoice_id' => $invoice['id'],
            'customer' => $invoice['customer'],
            'total' => $invoice['total'],
        ]);

        // Send receipt, extend access, update billing records
    }

    /**
     * Handle invoice payment failed.
     */
    protected function handleInvoicePaymentFailed(array $payload): void
    {
        parent::handleInvoicePaymentFailed($payload);

        $invoice = $payload['data']['object'];

        Log::warning('Invoice payment failed', [
            'invoice_id' => $invoice['id'],
            'customer' => $invoice['customer'],
            'attempt_count' => $invoice['attempt_count'],
        ]);

        // Notify user, trigger dunning flow
        // $user = $this->getUserByStripeId($invoice['customer']);
        // Mail::to($user)->send(new PaymentFailed($invoice));
    }

    /**
     * Handle charge refunded.
     */
    protected function handleChargeRefunded(array $payload): void
    {
        $charge = $payload['data']['object'];

        Log::info('Charge refunded', [
            'charge_id' => $charge['id'],
            'amount_refunded' => $charge['amount_refunded'],
        ]);

        // Update order status, notify customer
    }

    /**
     * Get user by Stripe customer ID.
     */
    protected function getUserByStripeId(string $stripeId)
    {
        return \App\Models\User::where('stripe_id', $stripeId)->first();
    }
}
```

## Step 4: Update Webhook Route

In `routes/web.php`, override Cashier's default webhook route:

```php
use App\Http\Controllers\WebhookController;

Route::post('/stripe/webhook', [WebhookController::class, 'handleWebhook'])
    ->name('cashier.webhook');
```

## Step 5: Exclude Webhook from CSRF Protection

In `bootstrap/app.php` or middleware configuration:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

Or in `app/Http/Middleware/VerifyCsrfToken.php`:

```php
protected $except = [
    'stripe/*',
];
```

## Step 6: Test Webhooks Locally

Install Stripe CLI:
```bash
# macOS
brew install stripe/stripe-cli/stripe

# Login
stripe login
```

Forward webhooks:
```bash
stripe listen --forward-to localhost:8000/stripe/webhook
```

Copy the webhook signing secret (`whsec_...`) to `.env`:
```env
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
```

Trigger test events:
```bash
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
stripe trigger invoice.payment_failed
```

## Step 7: Make Webhook Handlers Idempotent

Stripe may send the same event multiple times. Ensure handlers are idempotent:

```php
protected function handleInvoicePaymentSucceeded(array $payload): void
{
    $invoice = $payload['data']['object'];

    // Check if already processed
    if (Payment::where('stripe_invoice_id', $invoice['id'])->exists()) {
        Log::info('Invoice already processed, skipping', ['id' => $invoice['id']]);
        return;
    }

    // Process payment
    Payment::create([
        'stripe_invoice_id' => $invoice['id'],
        'user_id' => $this->getUserByStripeId($invoice['customer'])?->id,
        'amount' => $invoice['total'],
        'status' => 'paid',
    ]);

    parent::handleInvoicePaymentSucceeded($payload);
}
```

## Step 8: Verify Webhook in Production

After deploying, verify in Stripe Dashboard:
1. Go to **Developers** → **Webhooks**
2. Click your endpoint
3. Check **Recent events** for delivery status
4. Look for 200 response codes

</process>

<success_criteria>
Webhooks are working when:
- [ ] Webhook endpoint receives events from Stripe
- [ ] Signature verification passes (no 400 errors)
- [ ] Subscription events update database correctly
- [ ] Invoice events are logged
- [ ] Failed payments trigger notifications
- [ ] Handlers are idempotent (duplicate events handled)
- [ ] CSRF protection excludes webhook route
</success_criteria>

<anti_patterns>
Avoid:
- Processing webhooks without signature verification
- Assuming webhooks arrive in order (use timestamps)
- Not handling duplicate events (idempotency)
- Blocking webhook response with slow operations (use queues)
- Relying on redirect URLs instead of webhooks
</anti_patterns>
