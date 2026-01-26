<use_cases_overview>
Common Stripe implementation scenarios with recommended approaches. Use this as a starting point for planning your integration.
</use_cases_overview>

<saas_subscriptions>
## SaaS Subscriptions

**Scenario:** Monthly/yearly plans with tiered features.

**Components needed:**
- Products and Prices in Stripe Dashboard
- Checkout sessions for signup
- Customer portal for self-service
- Webhooks for subscription lifecycle
- Feature gating based on tier

**Implementation approach:**
```php
// config/stripe.php
'tiers' => [
    'free' => [
        'features' => ['5_projects', 'email_support'],
    ],
    'basic' => [
        'price_id' => 'price_basic_monthly',
        'features' => ['25_projects', 'email_support', 'api_access'],
    ],
    'pro' => [
        'price_id' => 'price_pro_monthly',
        'features' => ['unlimited_projects', 'priority_support', 'api_access', 'white_label'],
    ],
],

// Feature gating
public function store(Request $request)
{
    $user = $request->user();
    $projectCount = $user->projects()->count();

    $limits = [
        'free' => 5,
        'basic' => 25,
        'pro' => PHP_INT_MAX,
    ];

    if ($projectCount >= $limits[$user->subscriptionTier()]) {
        return back()->with('error', 'Upgrade to create more projects.');
    }

    // Create project...
}
```

**Webhooks to handle:**
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`
</saas_subscriptions>

<ecommerce_one_time>
## E-commerce (One-Time Purchases)

**Scenario:** Selling products with single payment.

**Components needed:**
- Checkout sessions with line items
- Webhooks for payment confirmation
- Order fulfillment system
- Inventory management

**Implementation approach:**
```php
// Create checkout for product purchase
public function checkout(Request $request)
{
    $cart = $request->user()->cart;

    $lineItems = $cart->items->map(fn ($item) => [
        'price_data' => [
            'currency' => 'usd',
            'product_data' => [
                'name' => $item->product->name,
                'images' => [$item->product->image_url],
            ],
            'unit_amount' => $item->product->price_cents,
        ],
        'quantity' => $item->quantity,
    ])->toArray();

    $session = Stripe::checkout->sessions->create([
        'line_items' => $lineItems,
        'mode' => 'payment',
        'success_url' => route('orders.success') . '?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('cart.index'),
        'metadata' => [
            'cart_id' => $cart->id,
            'user_id' => $request->user()->id,
        ],
    ]);

    return redirect($session->url);
}

// Webhook handler
public function handleCheckoutCompleted($session)
{
    $cart = Cart::find($session->metadata->cart_id);

    $order = Order::create([
        'user_id' => $session->metadata->user_id,
        'stripe_session_id' => $session->id,
        'status' => 'paid',
        'total' => $session->amount_total,
    ]);

    foreach ($cart->items as $item) {
        $order->items()->create([
            'product_id' => $item->product_id,
            'quantity' => $item->quantity,
            'price' => $item->product->price_cents,
        ]);

        // Decrement inventory
        $item->product->decrement('stock', $item->quantity);
    }

    $cart->delete();

    // Notify warehouse, send confirmation email, etc.
    OrderPlaced::dispatch($order);
}
```

**Webhooks to handle:**
- `checkout.session.completed`
- `payment_intent.succeeded`
- `charge.refunded`
</ecommerce_one_time>

<digital_products>
## Digital Products / Downloads

**Scenario:** Selling downloadable content, courses, or digital goods.

**Components needed:**
- Checkout sessions
- Access token generation
- Download link system
- License key management (optional)

**Implementation approach:**
```php
// After payment webhook
public function handlePaymentSuccess($session)
{
    $user = User::find($session->metadata->user_id);
    $productId = $session->metadata->product_id;

    // Grant access
    $purchase = Purchase::create([
        'user_id' => $user->id,
        'product_id' => $productId,
        'stripe_session_id' => $session->id,
    ]);

    // Generate secure download link
    $downloadToken = Str::random(64);
    $purchase->update(['download_token' => $downloadToken]);

    // Send download email
    Mail::to($user)->send(new DigitalProductPurchased($purchase));
}

// Download controller
public function download(Request $request, $token)
{
    $purchase = Purchase::where('download_token', $token)
        ->where('created_at', '>', now()->subDays(7)) // Link expires
        ->firstOrFail();

    $purchase->increment('download_count');

    return Storage::download($purchase->product->file_path);
}
```
</digital_products>

<marketplace>
## Marketplace (Connect)

**Scenario:** Platform where multiple sellers receive payments.

**Components needed:**
- Stripe Connect accounts for sellers
- Destination charges or separate charges
- Platform fee collection
- Seller onboarding flow

**Implementation approach:**
```php
// Seller onboarding
public function onboardSeller(Request $request)
{
    $user = $request->user();

    // Create Connect account
    $account = Stripe::accounts()->create([
        'type' => 'express',
        'email' => $user->email,
        'capabilities' => [
            'transfers' => ['requested' => true],
        ],
    ]);

    $user->update(['stripe_connect_id' => $account->id]);

    // Get onboarding link
    $link = Stripe::accountLinks()->create([
        'account' => $account->id,
        'refresh_url' => route('seller.onboard.refresh'),
        'return_url' => route('seller.onboard.complete'),
        'type' => 'account_onboarding',
    ]);

    return redirect($link->url);
}

// Checkout with platform fee
public function checkout(Request $request, Product $product)
{
    $seller = $product->seller;
    $platformFee = (int) ($product->price_cents * 0.10); // 10% platform fee

    $session = Stripe::checkout->sessions->create([
        'line_items' => [[
            'price_data' => [
                'currency' => 'usd',
                'product_data' => ['name' => $product->name],
                'unit_amount' => $product->price_cents,
            ],
            'quantity' => 1,
        ]],
        'mode' => 'payment',
        'payment_intent_data' => [
            'application_fee_amount' => $platformFee,
            'transfer_data' => [
                'destination' => $seller->stripe_connect_id,
            ],
        ],
        'success_url' => route('orders.success'),
        'cancel_url' => route('products.show', $product),
    ]);

    return redirect($session->url);
}
```

**Webhooks to handle:**
- `account.updated` (seller verification)
- `checkout.session.completed`
- `transfer.created`
</marketplace>

<metered_billing>
## Usage-Based / Metered Billing

**Scenario:** Charging based on API calls, storage, or resource usage.

**Components needed:**
- Metered price in Stripe
- Usage tracking system
- Usage reporting to Stripe
- Invoices with usage details

**Implementation approach:**
```php
// Track usage
public function trackApiCall(Request $request)
{
    $user = $request->user();

    // Store locally for batching
    ApiUsage::create([
        'user_id' => $user->id,
        'endpoint' => $request->path(),
        'timestamp' => now(),
    ]);
}

// Report usage to Stripe (scheduled job)
public function reportUsage()
{
    $users = User::whereHas('subscriptions')->get();

    foreach ($users as $user) {
        $subscription = $user->subscription('default');
        $subscriptionItem = $subscription->items->first();

        $usageCount = ApiUsage::where('user_id', $user->id)
            ->where('reported', false)
            ->count();

        if ($usageCount > 0) {
            Stripe::subscriptionItems()->createUsageRecord(
                $subscriptionItem->stripe_id,
                [
                    'quantity' => $usageCount,
                    'timestamp' => now()->timestamp,
                    'action' => 'increment',
                ]
            );

            ApiUsage::where('user_id', $user->id)
                ->where('reported', false)
                ->update(['reported' => true]);
        }
    }
}

// Create metered subscription
$user->newSubscription('default', 'price_metered_api')
    ->meteredPrice('price_metered_api')
    ->checkout();
```
</metered_billing>

<donations>
## Donations / Pay What You Want

**Scenario:** Variable amounts chosen by user.

**Implementation approach:**
```php
// Frontend: Let user choose amount
<form action="/donate" method="POST">
    <input type="number" name="amount" min="1" step="1" placeholder="Amount in USD">
    <button type="submit">Donate</button>
</form>

// Backend: Create session with custom amount
public function donate(Request $request)
{
    $request->validate([
        'amount' => 'required|numeric|min:1',
    ]);

    $amountCents = (int) ($request->amount * 100);

    $session = Stripe::checkout->sessions->create([
        'line_items' => [[
            'price_data' => [
                'currency' => 'usd',
                'product_data' => [
                    'name' => 'Donation',
                ],
                'unit_amount' => $amountCents,
            ],
            'quantity' => 1,
        ]],
        'mode' => 'payment',
        'success_url' => route('donate.thanks'),
        'cancel_url' => route('donate'),
    ]);

    return redirect($session->url);
}
```
</donations>

<invoicing>
## Manual Invoicing / B2B

**Scenario:** Send invoices for custom amounts, usually for business clients.

**Implementation approach:**
```php
// Create and send invoice
public function createInvoice(Request $request, Customer $customer)
{
    // Ensure customer exists in Stripe
    if (!$customer->stripe_id) {
        $stripeCustomer = Stripe::customers()->create([
            'email' => $customer->email,
            'name' => $customer->name,
        ]);
        $customer->update(['stripe_id' => $stripeCustomer->id]);
    }

    // Create invoice
    $invoice = Stripe::invoices()->create([
        'customer' => $customer->stripe_id,
        'collection_method' => 'send_invoice',
        'days_until_due' => 30,
    ]);

    // Add line items
    foreach ($request->items as $item) {
        Stripe::invoiceItems()->create([
            'customer' => $customer->stripe_id,
            'invoice' => $invoice->id,
            'description' => $item['description'],
            'amount' => $item['amount_cents'],
            'currency' => 'usd',
        ]);
    }

    // Finalize and send
    Stripe::invoices()->finalizeInvoice($invoice->id);
    Stripe::invoices()->sendInvoice($invoice->id);

    return back()->with('success', 'Invoice sent!');
}
```

**Webhooks to handle:**
- `invoice.sent`
- `invoice.paid`
- `invoice.payment_failed`
</invoicing>

<trials_and_freemium>
## Trials and Freemium

**Scenario:** Free tier with optional upgrade, or trial before commitment.

**Approaches:**

**1. Free tier (no payment method required):**
```php
// User just signs up, gets free features
// Upgrade requires checkout
public function upgrade(Request $request)
{
    return $request->user()
        ->newSubscription('default', $request->price_id)
        ->checkout();
}
```

**2. Trial with payment method:**
```php
$user->newSubscription('default', $priceId)
    ->trialDays(14)
    ->checkout(); // Payment method collected, not charged
```

**3. Trial without payment method:**
```php
$user->newSubscription('default', $priceId)
    ->trialDays(14)
    ->create(); // No payment method, requires Checkout later

// When trial ends:
if ($user->subscription('default')->onGracePeriod()) {
    // Prompt to add payment method
}
```

**4. Extended trial for certain users:**
```php
// Give sales leads longer trials
$user->newSubscription('default', $priceId)
    ->trialUntil(now()->addDays(30))
    ->checkout();
```
</trials_and_freemium>

<refunds>
## Refunds and Credits

**Scenario:** Customer requests money back or credit.

**Implementation approach:**
```php
// Full refund
public function refund(Order $order)
{
    $refund = Stripe::refunds()->create([
        'payment_intent' => $order->stripe_payment_intent_id,
    ]);

    $order->update(['status' => 'refunded']);

    return back()->with('success', 'Refund processed.');
}

// Partial refund
public function partialRefund(Order $order, Request $request)
{
    $refund = Stripe::refunds()->create([
        'payment_intent' => $order->stripe_payment_intent_id,
        'amount' => $request->amount_cents, // Partial amount
    ]);

    $order->update([
        'status' => 'partially_refunded',
        'refunded_amount' => $order->refunded_amount + $request->amount_cents,
    ]);
}

// Store credit instead of refund
public function issueCredit(Order $order)
{
    $user = $order->user;

    $user->increment('credit_balance', $order->total);

    $order->update(['status' => 'credited']);

    return back()->with('success', "Issued ${$order->total / 100} credit.");
}

// Apply credit at checkout
public function checkout(Request $request)
{
    $user = $request->user();
    $cartTotal = $this->calculateTotal($user->cart);

    $creditToApply = min($user->credit_balance, $cartTotal);
    $amountToCharge = $cartTotal - $creditToApply;

    if ($creditToApply > 0) {
        $user->decrement('credit_balance', $creditToApply);
    }

    if ($amountToCharge > 0) {
        // Charge remaining via Stripe
    } else {
        // Order is fully covered by credit
    }
}
```
</refunds>

<coupons_and_discounts>
## Coupons and Discounts

**Scenario:** Promotional pricing, referral codes, or loyalty discounts.

**Implementation:**
```php
// Create coupon in Stripe Dashboard or via API
$coupon = Stripe::coupons()->create([
    'percent_off' => 20,
    'duration' => 'once', // or 'forever', 'repeating'
    'id' => 'SUMMER20',
]);

// Apply coupon at checkout
$session = Stripe::checkout->sessions->create([
    'line_items' => [...],
    'mode' => 'subscription',
    'discounts' => [[
        'coupon' => 'SUMMER20',
    ]],
    'success_url' => route('billing.success'),
    'cancel_url' => route('billing.cancel'),
]);

// Or let user enter code
$session = Stripe::checkout->sessions->create([
    'line_items' => [...],
    'mode' => 'subscription',
    'allow_promotion_codes' => true, // User can enter code in Checkout
    'success_url' => route('billing.success'),
    'cancel_url' => route('billing.cancel'),
]);

// Referral program
public function applyReferralCredit(User $referrer, User $referred)
{
    // Give referrer credit on next invoice
    Stripe::customers()->createBalanceTransaction(
        $referrer->stripe_id,
        [
            'amount' => -1000, // $10 credit (negative = credit)
            'currency' => 'usd',
            'description' => "Referral credit for {$referred->email}",
        ]
    );
}
```
</coupons_and_discounts>

<multiple_products>
## Multiple Products in Cart

**Scenario:** User purchases several items at once.

**Implementation:**
```php
public function checkout(Request $request)
{
    $lineItems = collect($request->items)->map(function ($item) {
        $product = Product::find($item['id']);

        return [
            'price' => $product->stripe_price_id, // Use existing price
            'quantity' => $item['quantity'],
        ];

        // OR create price dynamically:
        return [
            'price_data' => [
                'currency' => 'usd',
                'product_data' => [
                    'name' => $product->name,
                    'metadata' => ['product_id' => $product->id],
                ],
                'unit_amount' => $product->price_cents,
            ],
            'quantity' => $item['quantity'],
        ];
    })->toArray();

    $session = Stripe::checkout->sessions->create([
        'line_items' => $lineItems,
        'mode' => 'payment',
        'success_url' => route('orders.success'),
        'cancel_url' => route('cart.index'),
    ]);

    return redirect($session->url);
}
```
</multiple_products>

<subscription_addons>
## Subscription Add-ons

**Scenario:** Base subscription with optional paid features.

**Implementation:**
```php
// Add extra item to subscription
public function addAddon(Request $request)
{
    $user = $request->user();
    $subscription = $user->subscription('default');

    // Add second price to existing subscription
    $subscription->addPrice('price_addon_extra_storage');

    return back()->with('success', 'Add-on activated.');
}

// Remove addon
public function removeAddon(Request $request)
{
    $user = $request->user();
    $subscription = $user->subscription('default');

    $subscription->removePrice('price_addon_extra_storage');

    return back()->with('success', 'Add-on removed.');
}

// Check addon status
public function hasAddon(User $user, string $addonPriceId): bool
{
    $subscription = $user->subscription('default');

    return $subscription?->items
        ->contains('stripe_price', $addonPriceId) ?? false;
}
```
</subscription_addons>
