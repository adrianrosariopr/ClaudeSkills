<overview>
Stripe Checkout is a prebuilt, hosted payment page that handles payment collection. It's PCI compliant, supports 3D Secure, and works on mobile. Always use Checkout instead of custom payment forms unless you have specific requirements.
</overview>

<why_use_checkout>
**Benefits:**
- PCI compliance (card data never touches your server)
- 3D Secure / SCA compliance built-in
- Mobile-optimized responsive design
- Multiple payment methods (cards, wallets, local methods)
- Automatic error handling and validation
- Localized in 30+ languages

**Trade-offs:**
- Less customization than custom forms
- Redirect away from your site
- Branding limited to logo and colors
</why_use_checkout>

<checkout_modes>
| Mode | Use Case | Creates |
|------|----------|---------|
| `payment` | One-time purchases | PaymentIntent |
| `subscription` | Recurring billing | Subscription + Invoice |
| `setup` | Save card for later | SetupIntent |
</checkout_modes>

<creating_sessions>
**Via Laravel Cashier (recommended):**
```php
// Subscription
return $user->newSubscription('default', $priceId)
    ->checkout([
        'success_url' => route('success') . '?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('cancel'),
    ]);

// One-time payment
return $user->checkout([$priceId => 1], [
    'success_url' => route('success'),
    'cancel_url' => route('cancel'),
]);
```

**Via Stripe SDK directly:**
```php
$session = \Stripe\Checkout\Session::create([
    'mode' => 'subscription',
    'customer' => $user->stripe_id,
    'line_items' => [
        [
            'price' => $priceId,
            'quantity' => 1,
        ],
    ],
    'success_url' => route('success') . '?session_id={CHECKOUT_SESSION_ID}',
    'cancel_url' => route('cancel'),
]);

return redirect($session->url);
```
</creating_sessions>

<session_options>
```php
\Stripe\Checkout\Session::create([
    // Required
    'mode' => 'subscription',           // payment, subscription, or setup
    'success_url' => '...',
    'cancel_url' => '...',
    'line_items' => [...],

    // Customer
    'customer' => 'cus_xxx',            // Existing customer
    'customer_email' => 'user@email',   // Pre-fill email (no customer)
    'customer_creation' => 'always',    // Create customer if none

    // Subscription options
    'subscription_data' => [
        'trial_period_days' => 7,
        'metadata' => ['plan' => 'pro'],
    ],

    // Payment options
    'payment_method_types' => ['card'], // Limit payment methods
    'allow_promotion_codes' => true,    // Enable coupon input

    // Customization
    'locale' => 'auto',                 // Language
    'submit_type' => 'pay',             // Button text: pay, book, donate
    'billing_address_collection' => 'required',

    // Metadata
    'metadata' => [
        'user_id' => $user->id,
        'order_id' => $orderId,
    ],
    'client_reference_id' => $orderId,  // Your reference ID
]);
```
</session_options>

<redirect_handling>
**Success URL variables:**
- `{CHECKOUT_SESSION_ID}` - Replaced with session ID

```php
'success_url' => route('success') . '?session_id={CHECKOUT_SESSION_ID}'
```

**On success page:**
```php
public function success(Request $request)
{
    $sessionId = $request->get('session_id');

    // Optionally retrieve session for display
    // But don't rely on this for fulfillment - use webhooks!
    $session = \Stripe\Checkout\Session::retrieve($sessionId);

    return view('success', ['session' => $session]);
}
```

**Important:** Never fulfill orders based solely on the success URL redirect. Always use webhooks.
</redirect_handling>

<webhook_events>
**Checkout-related events:**
| Event | Trigger |
|-------|---------|
| `checkout.session.completed` | Customer completed checkout |
| `checkout.session.async_payment_succeeded` | Async payment confirmed |
| `checkout.session.async_payment_failed` | Async payment failed |
| `checkout.session.expired` | Session expired (24h default) |

**After checkout.session.completed:**
- For `mode: payment` → Fulfill the order
- For `mode: subscription` → Wait for subscription webhooks
- For `mode: setup` → Card is saved for later

```php
// Webhook handler
protected function handleCheckoutSessionCompleted(array $payload)
{
    $session = $payload['data']['object'];

    if ($session['mode'] === 'payment') {
        // One-time payment - fulfill order
        $this->fulfillOrder($session);
    }
    // Subscriptions are handled by subscription webhooks
}
```
</webhook_events>

<customization>
**Branding (Stripe Dashboard → Settings → Branding):**
- Logo
- Icon
- Brand color
- Accent color
- Business name

**Per-session customization:**
```php
'custom_text' => [
    'submit' => [
        'message' => 'Your subscription will start immediately',
    ],
    'terms_of_service_acceptance' => [
        'message' => 'I agree to the [Terms of Service](https://example.com/terms)',
    ],
],
```
</customization>

<line_items>
**Using Price IDs (recommended):**
```php
'line_items' => [
    [
        'price' => 'price_xxxxx',
        'quantity' => 1,
    ],
],
```

**Inline pricing (one-off):**
```php
'line_items' => [
    [
        'price_data' => [
            'currency' => 'usd',
            'product_data' => [
                'name' => 'Custom Product',
                'description' => 'One-time purchase',
            ],
            'unit_amount' => 2000, // $20.00 in cents
        ],
        'quantity' => 1,
    ],
],
```

**Adjustable quantity:**
```php
'line_items' => [
    [
        'price' => 'price_xxxxx',
        'quantity' => 1,
        'adjustable_quantity' => [
            'enabled' => true,
            'minimum' => 1,
            'maximum' => 10,
        ],
    ],
],
```
</line_items>

<trials_and_discounts>
**Free trial:**
```php
// Via Cashier
$user->newSubscription('default', $priceId)
    ->trialDays(7)
    ->checkout([...]);

// Via Stripe SDK
'subscription_data' => [
    'trial_period_days' => 7,
],
```

**Coupon:**
```php
// Via Cashier
$user->newSubscription('default', $priceId)
    ->withCoupon('SAVE20')
    ->checkout([...]);

// Via Stripe SDK
'discounts' => [
    ['coupon' => 'SAVE20'],
],
```

**Promotion codes (customer-entered):**
```php
'allow_promotion_codes' => true,
```
</trials_and_discounts>

<guest_checkout>
For users without accounts:

```php
$session = \Stripe\Checkout\Session::create([
    'mode' => 'payment',
    'customer_email' => $request->email,  // Pre-fill email
    'customer_creation' => 'if_required', // Create customer for future use
    'line_items' => [...],
    'success_url' => '...',
    'cancel_url' => '...',
    'metadata' => [
        'email' => $request->email,
        'product' => $request->product,
    ],
]);
```

Fulfill via webhook using `customer_email` from session.
</guest_checkout>

<security_considerations>
- Never expose Price IDs that shouldn't be purchasable
- Validate price keys on the server side
- Use metadata to track what was purchased
- Always verify webhooks before fulfillment
- Don't trust client-side price/quantity data
</security_considerations>
