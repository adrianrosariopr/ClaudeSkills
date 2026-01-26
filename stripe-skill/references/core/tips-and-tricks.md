<tips_overview>
Lesser-known features, shortcuts, and practical tips for Stripe development.
</tips_overview>

<stripe_cli_tips>
## Stripe CLI Power Tips

**1. Listen to specific events only:**
```bash
# Only subscription events
stripe listen --events customer.subscription.created,customer.subscription.updated

# All invoice events
stripe listen --events 'invoice.*'
```

**2. Forward to multiple endpoints:**
```bash
stripe listen \
  --forward-to localhost:8000/stripe/webhook \
  --forward-connect-to localhost:8000/stripe/connect-webhook
```

**3. Trigger events with specific data:**
```bash
# Trigger with custom data
stripe trigger invoice.payment_failed \
  --override invoice:billing_reason=subscription_cycle

# Trigger subscription with trial
stripe trigger customer.subscription.created \
  --override subscription:trial_end=$(date -v+14d +%s)
```

**4. View recent events:**
```bash
stripe events list --limit 10

# Filter by type
stripe events list --type invoice.payment_failed
```

**5. Quick customer lookup:**
```bash
stripe customers retrieve cus_xxx
stripe subscriptions list --customer cus_xxx
stripe invoices list --customer cus_xxx
```

**6. Test webhook endpoint:**
```bash
# Resend a specific event
stripe events resend evt_xxx --webhook-endpoint we_xxx
```
</stripe_cli_tips>

<checkout_tricks>
## Checkout Session Tricks

**1. Custom success URL with session data:**
```php
// Pass data via success URL
'success_url' => route('checkout.success') . '?session_id={CHECKOUT_SESSION_ID}',

// Retrieve in controller
public function success(Request $request)
{
    $session = Stripe::checkout->sessions->retrieve(
        $request->query('session_id'),
        ['expand' => ['line_items', 'customer']]
    );

    // Access customer, line items, metadata
    $customerEmail = $session->customer_details->email;
}
```

**2. Pre-populate billing address:**
```php
$session = Stripe::checkout->sessions->create([
    'customer_update' => [
        'address' => 'auto', // Update customer with entered address
    ],
]);
```

**3. Collect custom fields:**
```php
$session = Stripe::checkout->sessions->create([
    'custom_fields' => [
        [
            'key' => 'company_name',
            'label' => ['type' => 'custom', 'custom' => 'Company Name'],
            'type' => 'text',
            'optional' => true,
        ],
        [
            'key' => 'vat_id',
            'label' => ['type' => 'custom', 'custom' => 'VAT ID'],
            'type' => 'text',
            'optional' => true,
        ],
    ],
]);

// Access in webhook
$companyName = $session->custom_fields[0]->text->value;
```

**4. Add terms of service checkbox:**
```php
$session = Stripe::checkout->sessions->create([
    'consent_collection' => [
        'terms_of_service' => 'required',
    ],
]);
```

**5. Automatic tax calculation:**
```php
$session = Stripe::checkout->sessions->create([
    'automatic_tax' => ['enabled' => true],
]);
// Requires Tax settings configured in Dashboard
```
</checkout_tricks>

<webhook_tricks>
## Webhook Tricks

**1. Get original object with expand:**
```php
// Webhook gives minimal data, expand for more
$event = Stripe::events()->retrieve($event->id, [
    'expand' => ['data.object.customer', 'data.object.subscription'],
]);
```

**2. Handle Connect webhooks separately:**
```php
// routes/web.php
Route::post('/stripe/webhook', [WebhookController::class, 'handle']);
Route::post('/stripe/connect', [ConnectWebhookController::class, 'handle']);

// Different webhook secrets
$event = Webhook::constructEvent(
    $payload,
    $signature,
    $isConnect ? config('stripe.connect_webhook_secret') : config('stripe.webhook_secret')
);
```

**3. Log webhook for debugging:**
```php
public function handle(Request $request)
{
    Log::channel('stripe')->info('Webhook received', [
        'type' => $request->input('type'),
        'id' => $request->input('id'),
    ]);

    // Process...
}
```

**4. Replay webhooks in development:**
```bash
# Get event ID from logs or dashboard
stripe events retrieve evt_xxx
stripe events resend evt_xxx
```

**5. Test webhook signature verification:**
```php
// In tests
public function test_rejects_invalid_signature()
{
    $response = $this->postJson('/stripe/webhook', ['type' => 'test'], [
        'Stripe-Signature' => 'invalid',
    ]);

    $response->assertStatus(400);
}
```
</webhook_tricks>

<customer_tricks>
## Customer Management Tricks

**1. Merge duplicate customers:**
```php
// Can't merge in Stripe, but can consolidate locally
public function mergeCustomers(User $primary, User $secondary)
{
    // Cancel secondary's subscriptions
    if ($secondary->subscribed()) {
        $secondary->subscription('default')->cancelNow();
    }

    // Transfer any balance
    if ($secondaryBalance = $this->getCustomerBalance($secondary)) {
        Stripe::customers()->createBalanceTransaction(
            $primary->stripe_id,
            ['amount' => $secondaryBalance, 'currency' => 'usd']
        );
    }

    // Update records
    Order::where('user_id', $secondary->id)->update(['user_id' => $primary->id]);

    $secondary->update(['stripe_id' => null]);
}
```

**2. Customer balance for credits:**
```php
// Add credit (negative amount = credit)
Stripe::customers()->createBalanceTransaction(
    $customer->stripe_id,
    ['amount' => -1000, 'currency' => 'usd', 'description' => 'Referral credit']
);

// Credit automatically applies to next invoice
```

**3. Default payment method management:**
```php
// Set default payment method
Stripe::customers()->update($customerId, [
    'invoice_settings' => [
        'default_payment_method' => 'pm_xxx',
    ],
]);

// List payment methods
$paymentMethods = Stripe::paymentMethods()->all([
    'customer' => $customerId,
    'type' => 'card',
]);
```

**4. Customer metadata for tracking:**
```php
Stripe::customers()->update($customerId, [
    'metadata' => [
        'user_id' => $user->id,
        'plan' => $user->plan,
        'signup_source' => 'organic',
        'first_purchase' => now()->toDateString(),
    ],
]);
```
</customer_tricks>

<subscription_tricks>
## Subscription Tricks

**1. Subscription schedules for future changes:**
```php
// Schedule upgrade in 30 days
$schedule = Stripe::subscriptionSchedules()->create([
    'from_subscription' => $subscription->stripe_id,
]);

Stripe::subscriptionSchedules()->update($schedule->id, [
    'phases' => [
        [
            'items' => [['price' => 'price_current', 'quantity' => 1]],
            'end_date' => now()->addDays(30)->timestamp,
        ],
        [
            'items' => [['price' => 'price_upgraded', 'quantity' => 1]],
        ],
    ],
]);
```

**2. Pause subscription billing:**
```php
// Pause collection (subscription stays active, no charges)
Stripe::subscriptions()->update($subscriptionId, [
    'pause_collection' => [
        'behavior' => 'void', // or 'keep_as_draft', 'mark_uncollectible'
    ],
]);

// Resume
Stripe::subscriptions()->update($subscriptionId, [
    'pause_collection' => '',
]);
```

**3. Anchor billing to specific day:**
```php
// Bill on 1st of each month
$subscription = Stripe::subscriptions()->create([
    'customer' => $customerId,
    'items' => [['price' => $priceId]],
    'billing_cycle_anchor' => Carbon::now()->startOfMonth()->addMonth()->timestamp,
    'proration_behavior' => 'create_prorations',
]);
```

**4. Multiple subscriptions per customer:**
```php
// User can have different subscription "types"
$user->newSubscription('main', 'price_main_plan')->create();
$user->newSubscription('addons', 'price_addon_storage')->create();

// Check each
$user->subscribed('main');
$user->subscribed('addons');
```

**5. Subscription metadata:**
```php
$subscription = $user->newSubscription('default', $priceId)
    ->withMetadata([
        'referral_code' => $referralCode,
        'campaign' => 'spring_sale',
    ])
    ->create();
```
</subscription_tricks>

<invoice_tricks>
## Invoice Tricks

**1. Add one-time charges to subscription invoice:**
```php
// Add item to upcoming invoice
Stripe::invoiceItems()->create([
    'customer' => $customerId,
    'amount' => 1000,
    'currency' => 'usd',
    'description' => 'Setup fee',
]);
// Will be included in next subscription invoice
```

**2. Preview upcoming invoice:**
```php
$upcomingInvoice = Stripe::invoices()->upcoming([
    'customer' => $customerId,
    'subscription' => $subscriptionId,
    // Optionally preview changes
    'subscription_items' => [
        ['id' => $itemId, 'price' => 'price_new'],
    ],
]);

// Show user what they'll pay
$nextCharge = $upcomingInvoice->amount_due;
$nextDate = Carbon::createFromTimestamp($upcomingInvoice->next_payment_attempt);
```

**3. Custom invoice numbering:**
```php
// Set in Stripe Dashboard or via API
Stripe::invoices()->update($invoiceId, [
    'custom_fields' => [
        ['name' => 'PO Number', 'value' => $poNumber],
    ],
]);
```

**4. Invoice memo/notes:**
```php
Stripe::invoices()->update($invoiceId, [
    'description' => 'Thank you for your business!',
    'footer' => 'Payment terms: Net 30',
]);
```

**5. Void vs refund:**
```php
// Void: Cancel unpaid invoice (no refund needed)
Stripe::invoices()->voidInvoice($invoiceId);

// Refund: Return money for paid invoice
Stripe::refunds()->create(['payment_intent' => $paymentIntentId]);
```
</invoice_tricks>

<testing_tricks>
## Testing Tricks

**1. Test clock for subscription testing:**
```php
// Create test clock
$testClock = Stripe::testHelpers()->testClocks()->create([
    'frozen_time' => now()->timestamp,
]);

// Create customer with test clock
$customer = Stripe::customers()->create([
    'test_clock' => $testClock->id,
    'email' => 'test@example.com',
]);

// Advance time
Stripe::testHelpers()->testClocks()->advance($testClock->id, [
    'frozen_time' => now()->addDays(30)->timestamp,
]);
// Subscription will renew!
```

**2. Specific test card behaviors:**
```bash
# Always succeeds
4242424242424242

# Always fails
4000000000000002

# Requires 3D Secure
4000002500003155

# Disputes/chargebacks
4000000000000259

# Specific decline codes
4000000000009995  # insufficient_funds
4000000000009987  # card_declined (lost_card)
```

**3. Test mode rate limits are generous:**
Test mode has higher rate limits - use it liberally for development.

**4. Inspect test mode in Dashboard:**
Toggle "Test mode" in Stripe Dashboard to see all test transactions, customers, etc.

**5. Clear test data:**
```bash
# Delete all test mode data (careful!)
stripe test_helpers test_clocks delete tc_xxx
```
</testing_tricks>

<debugging_tricks>
## Debugging Tricks

**1. Enable Stripe request logging:**
```php
// In development
\Stripe\Stripe::setEnableTelemetry(true);

// Custom logger
\Stripe\Stripe::setLogger(new class extends \Psr\Log\AbstractLogger {
    public function log($level, $message, array $context = [])
    {
        \Log::channel('stripe')->log($level, $message, $context);
    }
});
```

**2. Inspect webhook payloads:**
```php
public function handle(Request $request)
{
    Log::debug('Stripe webhook', [
        'headers' => $request->headers->all(),
        'payload' => $request->all(),
    ]);
}
```

**3. Check event type before parsing:**
```php
$eventType = $request->input('type');

// Don't process unneeded events
$handleEvents = [
    'invoice.paid',
    'customer.subscription.created',
];

if (!in_array($eventType, $handleEvents)) {
    return response('OK', 200);
}
```

**4. Use Stripe Dashboard > Developers > Events:**
- See all events
- View payloads
- Check webhook delivery status
- Retry failed webhooks

**5. idempotency keys for debugging:**
```php
$charge = Stripe::charges()->create([
    'amount' => 1000,
    'currency' => 'usd',
    'source' => $token,
], [
    'idempotency_key' => "charge_{$orderId}_{$attemptNumber}",
]);
// Retry with same key = same result (no duplicate charges)
```
</debugging_tricks>

<laravel_cashier_tips>
## Laravel Cashier Tips

**1. Override Cashier webhook handling:**
```php
// app/Http/Controllers/WebhookController.php
class WebhookController extends CashierController
{
    public function handleInvoicePaymentSucceeded($payload)
    {
        // Custom logic
        $this->sendThankYouEmail($payload);

        // Call parent
        return parent::handleInvoicePaymentSucceeded($payload);
    }
}
```

**2. Custom Stripe client options:**
```php
// AppServiceProvider
Cashier::calculateTaxes();

// Use specific API version
Cashier::useCustomerModel(Customer::class);
```

**3. Direct Stripe API access:**
```php
use Laravel\Cashier\Cashier;

$stripe = Cashier::stripe();
$products = $stripe->products->all(['active' => true]);
```

**4. Sync subscription from Stripe:**
```bash
# If local DB is out of sync
php artisan stripe:sync-subscriptions --all
```

**5. Subscription quantities:**
```php
// Seat-based pricing
$user->subscription('default')->updateQuantity(10);
$user->subscription('default')->incrementQuantity(5);
$user->subscription('default')->decrementQuantity(2);
```
</laravel_cashier_tips>

<dashboard_tips>
## Stripe Dashboard Tips

**1. Saved searches:**
- Create saved searches for common queries
- Filter by metadata, status, date ranges

**2. Developer shortcuts:**
- Press `g` then `c` to go to Customers
- Press `g` then `s` to go to Subscriptions
- Press `g` then `e` to go to Events

**3. Test mode toggle:**
- Top-right "Test mode" switch
- All test data is separate from live

**4. API logs:**
- Developers > Logs shows all API requests
- Filter by endpoint, status code, time

**5. Webhook logs:**
- Developers > Webhooks > Select endpoint
- See all deliveries, retry failed ones

**6. Radar rules:**
- Set up fraud protection rules
- Block suspicious transactions automatically
</dashboard_tips>

<quick_snippets>
## Quick Snippets

**Get customer's default payment method:**
```php
$pm = $user->defaultPaymentMethod();
$last4 = $pm?->card?->last4;
$brand = $pm?->card?->brand;
```

**Check if user ever subscribed:**
```php
$everSubscribed = $user->subscriptions()->exists();
```

**Get all invoices:**
```php
$invoices = $user->invoices();
$paidInvoices = $user->invoicesIncludingPending()->filter(fn($i) => $i->paid);
```

**Quick checkout redirect:**
```php
return $user->checkout('price_xxx', [
    'success_url' => route('success'),
    'cancel_url' => route('cancel'),
])->redirect();
```

**Check trial status:**
```php
$onTrial = $user->onTrial('default');
$trialEnds = $user->trialEndsAt('default');
$daysLeft = now()->diffInDays($trialEnds);
```

**Swap and invoice immediately:**
```php
$user->subscription('default')->swapAndInvoice('price_pro');
```

**Cancel at period end:**
```php
$user->subscription('default')->cancel(); // Cancels at end
$user->subscription('default')->cancelNow(); // Immediate
$user->subscription('default')->cancelNowAndInvoice(); // Immediate + final invoice
```

**Resume cancelled subscription:**
```php
if ($user->subscription('default')->onGracePeriod()) {
    $user->subscription('default')->resume();
}
```
</quick_snippets>
