<overview>
One-time payments are single purchases (not recurring). Use Stripe Checkout in `payment` mode for PCI-compliant collection of payment details.
</overview>

<checkout_vs_payment_intents>
| Method | Use Case | Complexity |
|--------|----------|------------|
| Checkout (`mode: payment`) | Simple purchases, digital goods | Low |
| Payment Intents | Custom UI, saved cards, complex flows | High |

**Recommendation:** Start with Checkout. Use Payment Intents only if you need custom UI.
</checkout_vs_payment_intents>

<creating_one_time_checkout>
**Via Laravel Cashier:**
```php
return $user->checkout([
    $priceId => $quantity,
], [
    'success_url' => route('checkout.success') . '?session_id={CHECKOUT_SESSION_ID}',
    'cancel_url' => route('checkout.cancel'),
    'metadata' => [
        'product' => $productKey,
        'user_id' => $user->id,
    ],
]);
```

**Via Stripe SDK (more control):**
```php
$session = \Stripe\Checkout\Session::create([
    'mode' => 'payment',
    'customer' => $user->stripe_id,  // Optional: link to customer
    'line_items' => [
        [
            'price' => $priceId,
            'quantity' => $quantity,
        ],
    ],
    'success_url' => route('checkout.success') . '?session_id={CHECKOUT_SESSION_ID}',
    'cancel_url' => route('checkout.cancel'),
    'metadata' => [
        'product' => $productKey,
        'user_id' => $user->id,
    ],
]);

return redirect($session->url);
```
</creating_one_time_checkout>

<guest_checkout>
For purchases without user accounts:

```php
$session = \Stripe\Checkout\Session::create([
    'mode' => 'payment',
    'customer_email' => $request->email,     // Pre-fill email
    'customer_creation' => 'if_required',    // Create customer record
    'line_items' => [
        ['price' => $priceId, 'quantity' => 1],
    ],
    'success_url' => route('checkout.success'),
    'cancel_url' => route('checkout.cancel'),
    'metadata' => [
        'email' => $request->email,
        'product' => $productKey,
    ],
]);
```

Fulfill via webhook using `customer_email` or metadata.
</guest_checkout>

<inline_pricing>
For dynamic pricing (not pre-created in Stripe):

```php
'line_items' => [
    [
        'price_data' => [
            'currency' => 'usd',
            'product_data' => [
                'name' => 'Custom Product',
                'description' => 'One-time purchase',
                'images' => ['https://example.com/product.jpg'],
            ],
            'unit_amount' => 2500,  // $25.00 in cents
        ],
        'quantity' => 1,
    ],
],
```

**Use for:**
- Dynamic pricing based on user input
- Custom configurations
- Donations with variable amounts
</inline_pricing>

<fulfillment>
**Never fulfill on success URL. Always use webhooks.**

```php
protected function handleCheckoutSessionCompleted(array $payload): void
{
    $session = $payload['data']['object'];

    // Only handle one-time payments
    if ($session['mode'] !== 'payment') {
        return;
    }

    // Idempotency check
    if (Purchase::where('stripe_session_id', $session['id'])->exists()) {
        return;
    }

    // Record purchase
    $purchase = Purchase::create([
        'stripe_session_id' => $session['id'],
        'stripe_payment_intent' => $session['payment_intent'],
        'user_id' => $session['metadata']['user_id'] ?? null,
        'product' => $session['metadata']['product'],
        'amount' => $session['amount_total'],
        'currency' => $session['currency'],
        'customer_email' => $session['customer_email'],
        'status' => 'completed',
    ]);

    // Grant access
    $this->grantProductAccess($purchase);

    // Send confirmation
    Mail::to($session['customer_email'])->send(new PurchaseConfirmation($purchase));
}

protected function grantProductAccess(Purchase $purchase): void
{
    if ($purchase->user_id) {
        $user = User::find($purchase->user_id);
        // Grant permission, unlock feature, etc.
        $user->givePermissionTo('access-' . $purchase->product);
    }
}
```
</fulfillment>

<purchase_model>
```php
// Migration
Schema::create('purchases', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->nullable()->constrained()->nullOnDelete();
    $table->string('stripe_session_id')->unique();
    $table->string('stripe_payment_intent')->nullable();
    $table->string('product');
    $table->integer('amount');
    $table->string('currency', 3)->default('usd');
    $table->string('customer_email')->nullable();
    $table->string('status')->default('pending');
    $table->timestamps();
});

// Model
class Purchase extends Model
{
    protected $fillable = [
        'user_id',
        'stripe_session_id',
        'stripe_payment_intent',
        'product',
        'amount',
        'currency',
        'customer_email',
        'status',
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function formattedAmount(): string
    {
        return '$' . number_format($this->amount / 100, 2);
    }
}
```
</purchase_model>

<multiple_items>
```php
'line_items' => [
    ['price' => $priceA, 'quantity' => 2],
    ['price' => $priceB, 'quantity' => 1],
],
```

All items in one checkout session = one payment.
</multiple_items>

<adjustable_quantity>
Let customer change quantity at checkout:

```php
'line_items' => [
    [
        'price' => $priceId,
        'quantity' => 1,
        'adjustable_quantity' => [
            'enabled' => true,
            'minimum' => 1,
            'maximum' => 10,
        ],
    ],
],
```
</adjustable_quantity>

<shipping>
For physical products:

```php
$session = \Stripe\Checkout\Session::create([
    'mode' => 'payment',
    'shipping_address_collection' => [
        'allowed_countries' => ['US', 'CA'],
    ],
    'shipping_options' => [
        [
            'shipping_rate_data' => [
                'type' => 'fixed_amount',
                'fixed_amount' => ['amount' => 500, 'currency' => 'usd'],
                'display_name' => 'Standard shipping',
                'delivery_estimate' => [
                    'minimum' => ['unit' => 'business_day', 'value' => 5],
                    'maximum' => ['unit' => 'business_day', 'value' => 7],
                ],
            ],
        ],
    ],
    // ...
]);
```

Access shipping in webhook:
```php
$shipping = $session['shipping_details'];
$address = $shipping['address'];
```
</shipping>

<refunds>
Process refunds via Stripe Dashboard or API:

```php
$refund = \Stripe\Refund::create([
    'payment_intent' => $paymentIntentId,
    'amount' => 1000,  // Partial refund in cents (optional)
]);
```

Webhook `charge.refunded` fires after refund.
</refunds>

<checking_purchase_status>
```php
// Check if user purchased a product
$user->purchases()->where('product', 'premium-template')->exists();

// Or via permissions (if granting on purchase)
$user->hasPermissionTo('access-premium-template');
```
</checking_purchase_status>

<common_patterns>
**Digital downloads:**
1. Create purchase record on webhook
2. Generate secure download link
3. Send email with link
4. Expire link after X downloads/time

**Feature unlocks:**
1. Create purchase record
2. Grant permission via Spatie
3. Check permission in middleware/gates

**Credits/tokens:**
1. Create purchase record
2. Add credits to user account
3. Deduct credits on usage
</common_patterns>
