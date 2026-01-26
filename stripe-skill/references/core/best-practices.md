<best_practices_overview>
Proven patterns and recommended approaches for Stripe integrations. These patterns prevent common issues and create maintainable payment systems.
</best_practices_overview>

<checkout_best_practices>
## Checkout Session Best Practices

**Always use Checkout Sessions for payments:**
```php
// GOOD - Stripe Checkout handles PCI, 3DS, and payment methods
$session = $user->newSubscription('default', $priceId)
    ->checkout([
        'success_url' => route('billing.success') . '?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('billing.cancel'),
    ]);

// BAD - Collecting card details on your own forms
// This requires PCI DSS SAQ D compliance (extensive security audit)
```

**Include session_id in success URL:**
```php
'success_url' => route('checkout.success') . '?session_id={CHECKOUT_SESSION_ID}'
```
This lets you verify the session was actually paid (not just someone visiting the URL).

**Always set cancel_url:**
```php
'cancel_url' => route('checkout.cancel')
```
Users need a way back if they abandon checkout.

**Use metadata for tracking:**
```php
$session = Stripe::checkout->sessions->create([
    'metadata' => [
        'user_id' => $user->id,
        'order_id' => $order->id,
        'campaign' => 'black_friday_2024',
    ],
]);
```
Metadata flows through to webhooks and Stripe Dashboard.
</checkout_best_practices>

<webhook_best_practices>
## Webhook Best Practices

**1. Always verify signatures:**
```php
try {
    $event = Webhook::constructEvent(
        $payload,
        $signature,
        config('services.stripe.webhook_secret')
    );
} catch (\Exception $e) {
    return response('Invalid signature', 400);
}
```

**2. Respond quickly (under 20 seconds):**
```php
// GOOD - Queue heavy processing
public function handleInvoicePaid($event)
{
    ProcessInvoicePaid::dispatch($event->data->object);
    return response('OK', 200); // Respond immediately
}

// BAD - Doing everything synchronously
public function handleInvoicePaid($event)
{
    $invoice = $event->data->object;
    $this->sendEmail($invoice);           // Slow
    $this->updateDatabase($invoice);       // Slow
    $this->notifySlack($invoice);          // Slow
    return response('OK', 200);            // Too late, Stripe times out
}
```

**3. Handle duplicate events:**
```php
public function handle($event)
{
    // Check if already processed
    if (ProcessedWebhook::where('stripe_event_id', $event->id)->exists()) {
        return response('Already processed', 200);
    }

    // Process and record
    $this->process($event);
    ProcessedWebhook::create(['stripe_event_id' => $event->id]);

    return response('OK', 200);
}
```

**4. Return 2xx even for unhandled events:**
```php
public function handle(Request $request)
{
    $event = $this->constructEvent($request);

    match ($event->type) {
        'invoice.paid' => $this->handleInvoicePaid($event),
        'customer.subscription.deleted' => $this->handleSubscriptionDeleted($event),
        default => null, // Don't throw errors for unhandled events
    };

    return response('OK', 200); // Always return 200
}
```
</webhook_best_practices>

<subscription_best_practices>
## Subscription Best Practices

**1. Use trial periods correctly:**
```php
// Trial without payment method (risky - may not convert)
$user->newSubscription('default', $priceId)
    ->trialDays(14)
    ->create();

// Trial with payment method (better conversion)
$user->newSubscription('default', $priceId)
    ->trialDays(14)
    ->checkout();
```

**2. Handle subscription states properly:**
```php
// Check actual subscription status
if ($user->subscribed('default')) {
    // Has active or trialing subscription
}

if ($user->subscription('default')?->onTrial()) {
    // Currently in trial
}

if ($user->subscription('default')?->onGracePeriod()) {
    // Cancelled but still has access until period end
}

if ($user->subscription('default')?->canceled()) {
    // Cancelled (may or may not have access)
}
```

**3. Prorate by default, but know when not to:**
```php
// Default: Prorate when changing plans
$user->subscription('default')->swap($newPriceId);

// No proration for downgrades (charge at next billing)
$user->subscription('default')->noProrate()->swap($newPriceId);

// Always prorate upgrades (immediate access, immediate charge)
$user->subscription('default')->swapAndInvoice($newPriceId);
```

**4. Store subscription data locally:**
```php
// Cache tier for fast access
$user->subscription_tier = $this->determineTier($subscription);
$user->save();

// Update via webhook, not API call
public function handleSubscriptionUpdated($subscription)
{
    $user = User::where('stripe_id', $subscription->customer)->first();
    $user->subscription_tier = $this->determineTier($subscription);
    $user->save();
}
```
</subscription_best_practices>

<error_handling_best_practices>
## Error Handling Best Practices

**1. Catch specific Stripe exceptions:**
```php
use Stripe\Exception\CardException;
use Stripe\Exception\InvalidRequestException;
use Stripe\Exception\AuthenticationException;
use Stripe\Exception\ApiConnectionException;
use Stripe\Exception\RateLimitException;

try {
    $charge = Stripe::charges()->create([...]);
} catch (CardException $e) {
    // Card declined - show user-friendly message
    return back()->with('error', $this->getCardErrorMessage($e));
} catch (RateLimitException $e) {
    // Too many requests - retry with backoff
    return $this->retryWithBackoff($operation);
} catch (InvalidRequestException $e) {
    // Invalid parameters - log and fix code
    Log::error('Stripe invalid request', ['error' => $e->getMessage()]);
    return back()->with('error', 'Something went wrong. Please try again.');
} catch (AuthenticationException $e) {
    // API key issue - alert developers
    Log::critical('Stripe authentication failed');
    return back()->with('error', 'Payment system temporarily unavailable.');
} catch (ApiConnectionException $e) {
    // Network issue - retry or queue
    return back()->with('error', 'Connection issue. Please try again.');
}
```

**2. User-friendly card error messages:**
```php
private function getCardErrorMessage(CardException $e): string
{
    return match ($e->getDeclineCode()) {
        'insufficient_funds' => 'Your card has insufficient funds.',
        'expired_card' => 'Your card has expired.',
        'incorrect_cvc' => 'The security code is incorrect.',
        'processing_error' => 'An error occurred processing your card. Please try again.',
        'card_declined' => 'Your card was declined. Please try a different payment method.',
        default => 'Unable to process your card. Please try a different payment method.',
    };
}
```

**3. Log Stripe errors with context:**
```php
catch (CardException $e) {
    Log::warning('Card declined', [
        'user_id' => $user->id,
        'decline_code' => $e->getDeclineCode(),
        'stripe_error_code' => $e->getStripeCode(),
        'last4' => $e->getSource()?->last4,
    ]);
}
```
</error_handling_best_practices>

<testing_best_practices>
## Testing Best Practices

**1. Use test mode completely separately:**
```env
# .env.local
STRIPE_KEY=pk_test_...
STRIPE_SECRET=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_test_...

# .env.production
STRIPE_KEY=pk_live_...
STRIPE_SECRET=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_live_...
```

**2. Create test products in test mode:**
Test and live modes have separate products/prices. Create matching products in both.

**3. Use test cards for specific scenarios:**
```php
// Success
'4242424242424242'

// Decline
'4000000000000002'

// Requires authentication (3DS)
'4000002500003155'

// Insufficient funds
'4000000000009995'
```

**4. Test webhooks locally with Stripe CLI:**
```bash
stripe listen --forward-to localhost:8000/stripe/webhook
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
```

**5. Mock Stripe in unit tests:**
```php
public function test_subscription_creation()
{
    $this->mock(StripeClient::class, function ($mock) {
        $mock->shouldReceive('subscriptions->create')
            ->once()
            ->andReturn(new Subscription(['id' => 'sub_test']));
    });

    $response = $this->post('/subscribe', ['price_id' => 'price_test']);
    $response->assertRedirect('/dashboard');
}
```
</testing_best_practices>

<security_best_practices>
## Security Best Practices

**1. Never expose secret keys:**
```php
// GOOD - Only publish publishable key
@vite(['resources/js/app.js'])
<script>
    window.stripeKey = "{{ config('services.stripe.key') }}"; // pk_...
</script>

// BAD - Never expose secret key
window.stripeSecret = "{{ config('services.stripe.secret') }}"; // sk_...
```

**2. Validate webhook signatures:**
```php
$signature = $request->header('Stripe-Signature');
$payload = $request->getContent();

// GOOD
$event = Webhook::constructEvent($payload, $signature, $webhookSecret);

// BAD - Never skip signature verification
$event = json_decode($payload); // Anyone can send fake webhooks
```

**3. Use restricted API keys for specific purposes:**
- Create read-only keys for reporting tools
- Create keys with only needed permissions for each service
- Rotate keys if compromised

**4. Verify ownership before actions:**
```php
public function cancelSubscription(Request $request, $subscriptionId)
{
    $subscription = Subscription::findOrFail($subscriptionId);

    // GOOD - Verify ownership
    if ($subscription->user_id !== auth()->id()) {
        abort(403);
    }

    $subscription->cancel();
}
```

**5. Rate limit payment endpoints:**
```php
// routes/web.php
Route::post('/subscribe', [SubscriptionController::class, 'store'])
    ->middleware(['auth', 'throttle:5,1']); // 5 attempts per minute
```
</security_best_practices>

<database_best_practices>
## Database Best Practices

**1. Store Stripe IDs for all objects:**
```php
Schema::table('users', function (Blueprint $table) {
    $table->string('stripe_id')->nullable()->index();
});

Schema::table('orders', function (Blueprint $table) {
    $table->string('stripe_payment_intent_id')->nullable();
    $table->string('stripe_checkout_session_id')->nullable();
});
```

**2. Never store full card numbers:**
```php
// GOOD - Store only references
$user->pm_type = $paymentMethod->card->brand;
$user->pm_last_four = $paymentMethod->card->last4;

// BAD - Never store card numbers
$user->card_number = $cardNumber; // PCI violation!
```

**3. Log important events:**
```php
Schema::create('payment_events', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id');
    $table->string('stripe_event_id')->unique();
    $table->string('type');
    $table->json('data');
    $table->timestamps();
});
```

**4. Track subscription changes:**
```php
Schema::create('subscription_history', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id');
    $table->string('action'); // created, upgraded, downgraded, cancelled, reactivated
    $table->string('from_price_id')->nullable();
    $table->string('to_price_id')->nullable();
    $table->timestamp('occurred_at');
});
```
</database_best_practices>

<frontend_best_practices>
## Frontend Best Practices

**1. Load Stripe.js from Stripe's CDN:**
```html
<!-- GOOD -->
<script src="https://js.stripe.com/v3/"></script>

<!-- BAD - Don't bundle Stripe.js -->
import Stripe from 'stripe'; // This is the server SDK!
```

**2. Initialize once:**
```javascript
// GOOD - Initialize once and reuse
const stripe = Stripe(window.stripeKey);

// BAD - Initializing multiple times
function handlePayment() {
    const stripe = Stripe(window.stripeKey); // Every click!
}
```

**3. Handle loading and error states:**
```jsx
function CheckoutButton({ priceId }) {
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);

    async function handleCheckout() {
        setLoading(true);
        setError(null);

        try {
            const response = await fetch('/api/checkout', {
                method: 'POST',
                body: JSON.stringify({ priceId }),
            });
            const { sessionId } = await response.json();
            await stripe.redirectToCheckout({ sessionId });
        } catch (e) {
            setError('Something went wrong. Please try again.');
        } finally {
            setLoading(false);
        }
    }

    return (
        <button onClick={handleCheckout} disabled={loading}>
            {loading ? 'Processing...' : 'Subscribe'}
        </button>
    );
}
```

**4. Confirm payment intent status after redirect:**
```javascript
// On success page
const sessionId = new URLSearchParams(window.location.search).get('session_id');
if (sessionId) {
    // Verify with backend, don't trust URL alone
    const response = await fetch(`/api/checkout/verify?session_id=${sessionId}`);
    const { status } = await response.json();
    if (status === 'complete') {
        showSuccessMessage();
    }
}
```
</frontend_best_practices>

<monitoring_best_practices>
## Monitoring Best Practices

**1. Track key metrics:**
- Failed payment rate
- Webhook processing time
- Subscription churn rate
- Revenue by tier

**2. Set up alerts:**
```php
// Alert on high failure rate
if ($failedPayments->last24Hours() > 10) {
    Notification::send($admins, new HighPaymentFailureRate());
}

// Alert on webhook processing delays
if ($webhookProcessingTime > 10000) { // 10 seconds
    Log::warning('Slow webhook processing', ['time' => $webhookProcessingTime]);
}
```

**3. Use Stripe Dashboard webhooks view:**
- Check webhook delivery status
- Retry failed webhooks
- View event payloads

**4. Log for debugging:**
```php
Log::info('Payment completed', [
    'user_id' => $user->id,
    'amount' => $charge->amount,
    'stripe_charge_id' => $charge->id,
]);
```
</monitoring_best_practices>
