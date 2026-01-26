<optimization_overview>
Strategies to reduce Stripe fees, improve conversion rates, and optimize payment flows.
</optimization_overview>

<reduce_processing_fees>
## Reducing Processing Fees

**1. Use ACH for US customers (0.8% capped at $5):**
```php
$session = Stripe::checkout->sessions->create([
    'payment_method_types' => ['card', 'us_bank_account'],
    // ACH: 0.8% capped at $5
    // Card: 2.9% + $0.30
    // For a $500 payment: ACH = $4, Card = $14.80
]);
```

**2. Enable Link for faster checkout:**
```php
$session = Stripe::checkout->sessions->create([
    'payment_method_types' => ['card', 'link'],
    // Link reduces friction, which improves conversion
]);
```

**3. Offer annual billing (one transaction instead of 12):**
```php
// Monthly: 12 × ($10 × 2.9% + $0.30) = $7.08/year in fees
// Annual: 1 × ($120 × 2.9% + $0.30) = $3.78/year in fees
// Savings: 47%

'prices' => [
    'monthly' => 'price_monthly',
    'annual' => 'price_annual', // Often discounted 10-20% too
],
```

**4. Use regional pricing (local cards have lower fees):**
```php
// Cards issued in same country as your Stripe account = lower fees
// Consider Stripe accounts in multiple regions for large international business
```

**5. Avoid unnecessary refunds (you don't get the fee back):**
```php
// If refunding, you still pay the original processing fee
// Consider store credit for small amounts
if ($refundAmount < 1000) { // Under $10
    // Offer store credit instead
    $user->increment('credit_balance', $refundAmount);
}
```

**6. Batch usage reporting for metered billing:**
```php
// BAD - One API call per usage event
foreach ($usageEvents as $event) {
    Stripe::subscriptionItems()->createUsageRecord(...);
}

// GOOD - Aggregate and report periodically
$totalUsage = UsageEvent::where('reported', false)->count();
Stripe::subscriptionItems()->createUsageRecord(
    $subscriptionItemId,
    ['quantity' => $totalUsage, 'action' => 'increment']
);
```
</reduce_processing_fees>

<improve_conversion>
## Improving Checkout Conversion

**1. Reduce steps to checkout:**
```php
// GOOD - Direct to checkout
return redirect($session->url);

// BAD - Extra confirmation page before checkout
return view('confirm-before-checkout');
```

**2. Pre-fill customer information:**
```php
$session = Stripe::checkout->sessions->create([
    'customer_email' => $user->email,
    // Or use existing customer
    'customer' => $user->stripe_id,
]);
```

**3. Use Stripe Checkout (not custom forms):**
- Higher conversion than custom forms
- Automatically optimized for mobile
- Supports multiple payment methods

**4. Offer multiple payment methods:**
```php
$session = Stripe::checkout->sessions->create([
    'payment_method_types' => [
        'card',
        'link',           // Fast checkout
        'us_bank_account', // ACH
        'afterpay_clearpay', // Buy now pay later
    ],
]);
```

**5. Clear pricing display:**
```php
// Show what they'll pay before checkout
<div>
    <span>Pro Plan</span>
    <span>$29/month</span>
    <small>Billed monthly. Cancel anytime.</small>
</div>
```

**6. Add trust signals:**
```html
<div class="trust-signals">
    <img src="/secure-payment.svg" alt="Secure payment" />
    <span>Powered by Stripe</span>
    <span>256-bit encryption</span>
</div>
```

**7. Reduce price shock:**
```php
// Show price before tax, or include tax in displayed price
// Show savings for annual plans
$annualSavings = ($monthlyPrice * 12) - $annualPrice;
```
</improve_conversion>

<reduce_churn>
## Reducing Subscription Churn

**1. Implement Smart Retries:**
Stripe automatically retries failed payments, but configure it well:
- Dashboard → Settings → Billing → Automatic collection
- Set retry schedule (e.g., 3 retries over 7 days)

**2. Send dunning emails:**
```php
// Webhook: invoice.payment_failed
public function handlePaymentFailed($invoice)
{
    $user = User::where('stripe_id', $invoice->customer)->first();

    // First failure: gentle reminder
    if ($invoice->attempt_count === 1) {
        Mail::to($user)->send(new PaymentFailedFirstAttempt($invoice));
    }

    // Second failure: urgent
    if ($invoice->attempt_count === 2) {
        Mail::to($user)->send(new PaymentFailedUrgent($invoice));
    }

    // Final attempt: last chance
    if ($invoice->attempt_count >= 3) {
        Mail::to($user)->send(new PaymentFailedFinal($invoice));
    }
}
```

**3. Allow card updates before cancellation:**
```php
// Webhook: customer.subscription.updated (status = past_due)
public function handlePastDue($subscription)
{
    $user = User::where('stripe_id', $subscription->customer)->first();

    // Send link to update payment method
    $portalSession = $user->billingPortalSession(route('dashboard'));
    Mail::to($user)->send(new UpdatePaymentMethod($portalSession->url));
}
```

**4. Offer downgrade instead of cancel:**
```php
// In cancellation flow
public function showCancelOptions(Request $request)
{
    return view('billing.cancel-options', [
        'downgradeTier' => $this->getCheaperTier($request->user()),
        'pauseOption' => true, // Pause billing temporarily
    ]);
}
```

**5. Implement cancellation survey:**
```php
public function cancel(Request $request)
{
    $reason = $request->input('reason');

    CancellationReason::create([
        'user_id' => $request->user()->id,
        'reason' => $reason,
    ]);

    // If reason is price, offer discount
    if ($reason === 'too_expensive') {
        return $this->offerDiscount($request->user());
    }

    $request->user()->subscription('default')->cancel();
}
```

**6. Win-back campaigns:**
```php
// Scheduled job for users cancelled 30 days ago
$cancelledUsers = User::whereHas('subscriptions', function ($q) {
    $q->where('ends_at', '<', now()->subDays(30))
      ->where('ends_at', '>', now()->subDays(37));
})->get();

foreach ($cancelledUsers as $user) {
    Mail::to($user)->send(new WinBackOffer(
        discountCode: 'COMEBACK20',
        validDays: 7
    ));
}
```
</reduce_churn>

<optimize_webhooks>
## Optimizing Webhook Processing

**1. Respond immediately, process async:**
```php
public function handle(Request $request)
{
    $event = $this->verifySignature($request);

    // Queue for processing
    ProcessStripeWebhook::dispatch($event);

    // Respond immediately
    return response('OK', 200);
}
```

**2. Batch database operations:**
```php
// BAD - Individual updates
foreach ($subscriptions as $sub) {
    User::where('stripe_id', $sub->customer)
        ->update(['subscription_status' => $sub->status]);
}

// GOOD - Batch update
$updates = collect($subscriptions)->mapWithKeys(fn ($sub) => [
    $sub->customer => $sub->status
]);

User::whereIn('stripe_id', $updates->keys())
    ->each(fn ($user) => $user->update([
        'subscription_status' => $updates[$user->stripe_id]
    ]));
```

**3. Cache Stripe customer data:**
```php
public function getStripeCustomer(User $user)
{
    return Cache::remember(
        "stripe_customer:{$user->stripe_id}",
        now()->addMinutes(5),
        fn () => Stripe::customers()->retrieve($user->stripe_id)
    );
}

// Invalidate cache on webhook
public function handleCustomerUpdated($customer)
{
    Cache::forget("stripe_customer:{$customer->id}");
}
```

**4. Handle events idempotently:**
```php
public function process($event)
{
    // Check if already processed
    $processed = ProcessedEvent::where('stripe_event_id', $event->id)->exists();
    if ($processed) {
        return;
    }

    // Process
    $this->handleEvent($event);

    // Record
    ProcessedEvent::create(['stripe_event_id' => $event->id]);
}
```
</optimize_webhooks>

<api_efficiency>
## API Call Efficiency

**1. Use expand to reduce API calls:**
```php
// BAD - 3 API calls
$customer = Stripe::customers()->retrieve($customerId);
$subscription = Stripe::subscriptions()->retrieve($customer->subscriptions->data[0]->id);
$invoices = Stripe::invoices()->all(['customer' => $customerId]);

// GOOD - 1 API call
$customer = Stripe::customers()->retrieve($customerId, [
    'expand' => ['subscriptions', 'invoices'],
]);
```

**2. Use list with filters instead of retrieving individually:**
```php
// BAD - Multiple retrieve calls
foreach ($customerIds as $id) {
    $customers[] = Stripe::customers()->retrieve($id);
}

// GOOD - Single list call with filter
$customers = Stripe::customers()->all([
    'limit' => 100,
    // Filter if possible
]);
```

**3. Handle rate limits gracefully:**
```php
use Stripe\Exception\RateLimitException;

public function makeStripeCall($operation, $retries = 3)
{
    for ($i = 0; $i < $retries; $i++) {
        try {
            return $operation();
        } catch (RateLimitException $e) {
            if ($i === $retries - 1) throw $e;
            sleep(pow(2, $i)); // Exponential backoff: 1s, 2s, 4s
        }
    }
}
```

**4. Cache static data:**
```php
// Products and prices rarely change
$products = Cache::remember('stripe_products', now()->addHour(), function () {
    return Stripe::products()->all(['active' => true]);
});

$prices = Cache::remember('stripe_prices', now()->addHour(), function () {
    return Stripe::prices()->all(['active' => true]);
});
```
</api_efficiency>

<database_optimization>
## Database Optimization

**1. Index Stripe IDs:**
```php
Schema::table('users', function (Blueprint $table) {
    $table->index('stripe_id');
});

Schema::table('subscriptions', function (Blueprint $table) {
    $table->index('stripe_id');
    $table->index('stripe_status');
});
```

**2. Store computed subscription status locally:**
```php
// Instead of calling Stripe every time
public function subscriptionTier(): string
{
    // GOOD - Read from local database
    return $this->cached_subscription_tier ?? 'free';
}

// Update via webhook
public function handleSubscriptionUpdated($subscription)
{
    User::where('stripe_id', $subscription->customer)
        ->update(['cached_subscription_tier' => $this->mapTier($subscription)]);
}
```

**3. Eager load subscription data:**
```php
// BAD - N+1 queries
@foreach ($users as $user)
    {{ $user->subscription('default')?->stripe_status }}
@endforeach

// GOOD - Eager load
$users = User::with('subscriptions')->get();
```
</database_optimization>

<cost_tracking>
## Cost Tracking and Analytics

**1. Track Stripe fees:**
```php
// Webhook: charge.succeeded
public function handleChargeSucceeded($charge)
{
    $balanceTransaction = Stripe::balanceTransactions()->retrieve(
        $charge->balance_transaction
    );

    PaymentRecord::create([
        'stripe_charge_id' => $charge->id,
        'amount' => $charge->amount,
        'fee' => $balanceTransaction->fee,
        'net' => $balanceTransaction->net,
        'fee_details' => json_encode($balanceTransaction->fee_details),
    ]);
}
```

**2. Calculate effective fee rates:**
```php
public function getEffectiveFeeRate(Carbon $start, Carbon $end): float
{
    $payments = PaymentRecord::whereBetween('created_at', [$start, $end])->get();

    $totalAmount = $payments->sum('amount');
    $totalFees = $payments->sum('fee');

    return ($totalFees / $totalAmount) * 100; // Percentage
}
```

**3. Compare pricing strategies:**
```php
// Annual vs monthly revenue analysis
$monthlyRevenue = Subscription::where('stripe_price', 'price_monthly')
    ->sum('amount');
$annualRevenue = Subscription::where('stripe_price', 'price_annual')
    ->sum('amount');

$monthlyFees = $monthlySubscriberCount * 12 * ($monthlyPrice * 0.029 + 0.30);
$annualFees = $annualSubscriberCount * ($annualPrice * 0.029 + 0.30);
```
</cost_tracking>

<performance_metrics>
## Key Performance Metrics

Track these metrics for a healthy payment system:

**1. Conversion Rate:**
```php
$checkoutStarted = AnalyticsEvent::where('event', 'checkout_started')->count();
$checkoutCompleted = AnalyticsEvent::where('event', 'checkout_completed')->count();
$conversionRate = ($checkoutCompleted / $checkoutStarted) * 100;
```

**2. Failed Payment Rate:**
```php
$totalAttempts = Invoice::count();
$failedAttempts = Invoice::where('status', 'open')->count();
$failedRate = ($failedAttempts / $totalAttempts) * 100;
// Healthy: under 5%
```

**3. Churn Rate:**
```php
$startOfMonth = now()->startOfMonth();
$subscribersAtStart = Subscription::where('created_at', '<', $startOfMonth)
    ->where('stripe_status', 'active')
    ->count();
$cancelledThisMonth = Subscription::where('ends_at', '>=', $startOfMonth)
    ->where('ends_at', '<', now())
    ->count();
$churnRate = ($cancelledThisMonth / $subscribersAtStart) * 100;
// Healthy: under 5% monthly
```

**4. Average Revenue Per User (ARPU):**
```php
$totalRevenue = Invoice::where('status', 'paid')
    ->whereMonth('created_at', now()->month)
    ->sum('amount_paid');
$activeSubscribers = Subscription::where('stripe_status', 'active')->count();
$arpu = $totalRevenue / $activeSubscribers;
```

**5. Lifetime Value (LTV):**
```php
$avgSubscriptionLength = Subscription::avg(
    DB::raw('DATEDIFF(COALESCE(ends_at, NOW()), created_at)')
);
$avgMonthlyRevenue = $arpu;
$ltv = ($avgSubscriptionLength / 30) * $avgMonthlyRevenue;
```
</performance_metrics>
