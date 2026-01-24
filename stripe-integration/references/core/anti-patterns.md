<overview>
Common mistakes when implementing Stripe payments with Laravel. Avoid these patterns to ensure reliable, secure payment processing.
</overview>

<anti_pattern name="relying-on-success-url">
**Problem:** Fulfilling orders based on success URL redirect.

```php
// BAD
public function success(Request $request)
{
    // User reached success page, must have paid!
    $user->givePermissionTo('premium');
    Order::create(['status' => 'completed']);
}
```

**Why it's wrong:**
- User can close browser before redirect
- User can manually navigate to success URL
- Network issues can prevent redirect
- Bots can hit your success URL

**Fix:** Always use webhooks for fulfillment.

```php
// GOOD: Webhook handler
protected function handleCheckoutSessionCompleted(array $payload)
{
    $session = $payload['data']['object'];
    // Verify payment, then fulfill
}

// Success page is just a "thank you" message
public function success()
{
    return view('checkout.success');
}
```
</anti_pattern>

<anti_pattern name="exposing-api-keys">
**Problem:** Committing API keys or exposing them.

```php
// BAD: Hardcoded keys
\Stripe\Stripe::setApiKey('sk_live_xxxxx');

// BAD: In version control
// config/services.php
'stripe' => [
    'secret' => 'sk_live_xxxxx',
]
```

**Fix:** Environment variables only.

```php
// GOOD
// .env (gitignored)
STRIPE_SECRET=sk_live_xxxxx

// config/services.php
'stripe' => [
    'secret' => env('STRIPE_SECRET'),
]
```
</anti_pattern>

<anti_pattern name="trusting-client-data">
**Problem:** Using client-provided price IDs or amounts.

```php
// BAD: Client sends price ID directly
public function checkout(Request $request)
{
    return $user->checkout($request->price_id);
}
```

**Why it's wrong:**
- Attacker can use any price ID
- Could checkout with $0.01 price
- No validation of what they're buying

**Fix:** Map client keys to server-side values.

```php
// GOOD
public function checkout(Request $request)
{
    $request->validate(['price' => 'required|in:basic_monthly,pro_monthly']);

    $prices = [
        'basic_monthly' => config('stripe.prices.basic_monthly'),
        'pro_monthly' => config('stripe.prices.pro_monthly'),
    ];

    return $user->checkout($prices[$request->price]);
}
```
</anti_pattern>

<anti_pattern name="missing-idempotency">
**Problem:** Processing webhooks without idempotency checks.

```php
// BAD
protected function handleInvoicePaymentSucceeded(array $payload)
{
    $invoice = $payload['data']['object'];

    // What if this webhook is sent twice?
    Payment::create([...]);
    Mail::send(new Receipt($invoice));
}
```

**Fix:** Check if already processed.

```php
// GOOD
protected function handleInvoicePaymentSucceeded(array $payload)
{
    $invoice = $payload['data']['object'];

    if (Payment::where('stripe_invoice_id', $invoice['id'])->exists()) {
        Log::info('Already processed', ['id' => $invoice['id']]);
        return;
    }

    Payment::create(['stripe_invoice_id' => $invoice['id'], ...]);
    Mail::send(new Receipt($invoice));
}
```
</anti_pattern>

<anti_pattern name="skipping-signature-verification">
**Problem:** Processing webhooks without verifying signatures.

```php
// BAD
public function handleWebhook(Request $request)
{
    $event = $request->all();
    // Process event without verification
}
```

**Why it's wrong:**
- Anyone can POST fake events
- Attackers can trigger fake payments
- No guarantee event is from Stripe

**Fix:** Always verify signatures (Cashier does this automatically).

```php
// GOOD (using Cashier)
class WebhookController extends CashierController
{
    // Cashier verifies signatures before calling handlers
}

// GOOD (manual)
$event = \Stripe\Webhook::constructEvent(
    $payload, $sigHeader, $secret
);
```
</anti_pattern>

<anti_pattern name="building-custom-card-forms">
**Problem:** Creating custom forms to collect card details.

```html
<!-- BAD -->
<form>
    <input name="card_number" />
    <input name="cvv" />
    <input name="expiry" />
</form>
```

**Why it's wrong:**
- PCI compliance nightmare
- Security liability
- More work to maintain

**Fix:** Use Stripe Checkout or Elements.

```php
// GOOD: Stripe Checkout
return $user->checkout($priceId, [
    'success_url' => route('success'),
    'cancel_url' => route('cancel'),
]);
```
</anti_pattern>

<anti_pattern name="mixing-test-and-live">
**Problem:** Using test keys with live mode or vice versa.

```env
# BAD: Test key in production
STRIPE_KEY=pk_test_xxxxx
STRIPE_SECRET=sk_test_xxxxx
```

**Symptoms:**
- "No such price" errors
- Customers not found
- Webhooks failing

**Fix:** Use correct keys for each environment.

```env
# .env.production
STRIPE_KEY=pk_live_xxxxx
STRIPE_SECRET=sk_live_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_live_xxxxx
```
</anti_pattern>

<anti_pattern name="ignoring-grace-periods">
**Problem:** Immediately revoking access on cancellation.

```php
// BAD
public function cancelSubscription()
{
    $user->subscription('default')->cancelNow();
    $user->revokePermissionTo('premium');
}
```

**Why it's wrong:**
- Customer paid for full period
- Bad user experience
- Potential disputes

**Fix:** Respect grace period.

```php
// GOOD
public function cancelSubscription()
{
    $user->subscription('default')->cancel(); // At period end
}

// Access check
public function premiumFeature()
{
    if ($user->subscribed('default') || $user->subscription('default')?->onGracePeriod()) {
        // Allow access
    }
}
```
</anti_pattern>

<anti_pattern name="hardcoding-price-ids">
**Problem:** Hardcoding Stripe IDs throughout the codebase.

```php
// BAD
$user->newSubscription('default', 'price_1234567890')
    ->checkout([...]);
```

**Why it's wrong:**
- IDs differ between test/live
- Hard to update prices
- No single source of truth

**Fix:** Centralize in config.

```php
// config/stripe.php
'prices' => [
    'basic_monthly' => env('STRIPE_PRICE_BASIC_MONTHLY'),
]

// Usage
$user->newSubscription('default', config('stripe.prices.basic_monthly'))
```
</anti_pattern>

<anti_pattern name="blocking-webhook-response">
**Problem:** Doing slow operations in webhook handler.

```php
// BAD
protected function handleCheckoutSessionCompleted(array $payload)
{
    // Slow API call
    $this->generatePDF($order);

    // Slow email sending
    Mail::send(new OrderConfirmation($order));

    // Long database operations
    $this->updateInventory($order);
}
```

**Why it's wrong:**
- Stripe times out (5-second limit)
- Webhook marked as failed
- Retry loop begins

**Fix:** Queue slow operations.

```php
// GOOD
protected function handleCheckoutSessionCompleted(array $payload)
{
    // Quick: Record essential data
    $order = Order::create([...]);

    // Queue: Slow operations
    dispatch(new ProcessOrder($order));

    // Return quickly
}
```
</anti_pattern>

<anti_pattern name="storing-card-details">
**Problem:** Attempting to store card details yourself.

```php
// BAD
$user->update([
    'card_number' => $request->card_number, // NEVER do this
    'card_cvv' => $request->cvv,
]);
```

**Why it's wrong:**
- PCI compliance violation
- Massive security liability
- Potential legal issues

**Fix:** Let Stripe handle card storage. Use `pm_last_four` for display only.
</anti_pattern>

<anti_pattern name="not-logging-webhook-errors">
**Problem:** Silently failing on webhook errors.

```php
// BAD
protected function handleInvoicePaymentFailed(array $payload)
{
    try {
        // Process
    } catch (\Exception $e) {
        // Silent failure
    }
}
```

**Fix:** Log errors for debugging.

```php
// GOOD
protected function handleInvoicePaymentFailed(array $payload)
{
    try {
        // Process
    } catch (\Exception $e) {
        Log::error('Webhook processing failed', [
            'event_id' => $payload['id'],
            'error' => $e->getMessage(),
        ]);
        throw $e; // Return 500 so Stripe retries
    }
}
```
</anti_pattern>

<checklist>
**Before shipping, verify you're NOT:**
- [ ] Fulfilling orders on success URL alone
- [ ] Exposing API keys in code or logs
- [ ] Trusting client-provided price/amount data
- [ ] Processing webhooks without idempotency
- [ ] Skipping webhook signature verification
- [ ] Building custom card collection forms
- [ ] Mixing test and live credentials
- [ ] Ignoring subscription grace periods
- [ ] Hardcoding Stripe IDs throughout code
- [ ] Blocking webhook response with slow operations
- [ ] Storing any card details yourself
- [ ] Silently swallowing webhook errors
</checklist>
