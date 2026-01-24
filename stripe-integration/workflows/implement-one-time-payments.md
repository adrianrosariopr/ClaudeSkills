# Workflow: Implement One-Time Payments

<required_reading>
**Read these reference files NOW:**
1. references/one-time-payments.md
2. references/stripe-checkout.md
</required_reading>

<process>

## Step 1: Create Product in Stripe Dashboard

1. Go to Stripe Dashboard → **Products**
2. Click **+ Add product**
3. Select **One-time** pricing
4. Enter product details and price
5. Copy the **Price ID** (`price_...`)

Add to `.env`:
```env
STRIPE_PRICE_PRODUCT_A=price_xxxxx
STRIPE_PRICE_PRODUCT_B=price_xxxxx
```

Update `config/stripe.php`:
```php
'products' => [
    'product_a' => env('STRIPE_PRICE_PRODUCT_A'),
    'product_b' => env('STRIPE_PRICE_PRODUCT_B'),
],
```

## Step 2: Create Checkout Controller

Create `app/Http/Controllers/CheckoutController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class CheckoutController extends Controller
{
    /**
     * Create checkout session for one-time purchase.
     */
    public function checkout(Request $request)
    {
        $request->validate([
            'product' => 'required|string',
            'quantity' => 'integer|min:1|max:10',
        ]);

        $priceId = $this->resolvePriceId($request->product);
        $quantity = $request->quantity ?? 1;

        return $request->user()->checkout([
            $priceId => $quantity,
        ], [
            'success_url' => route('checkout.success') . '?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout.cancel'),
            'metadata' => [
                'product' => $request->product,
                'user_id' => $request->user()->id,
            ],
        ]);
    }

    /**
     * Guest checkout (no authentication required).
     */
    public function guestCheckout(Request $request)
    {
        $request->validate([
            'product' => 'required|string',
            'email' => 'required|email',
        ]);

        $priceId = $this->resolvePriceId($request->product);

        $checkout = \Stripe\Checkout\Session::create([
            'line_items' => [
                [
                    'price' => $priceId,
                    'quantity' => 1,
                ],
            ],
            'mode' => 'payment',
            'customer_email' => $request->email,
            'success_url' => route('checkout.success') . '?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout.cancel'),
            'metadata' => [
                'product' => $request->product,
                'email' => $request->email,
            ],
        ]);

        return redirect($checkout->url);
    }

    /**
     * Handle successful checkout.
     */
    public function success(Request $request)
    {
        $sessionId = $request->get('session_id');

        if ($sessionId) {
            // Optionally retrieve session to show details
            // Real fulfillment should happen via webhook
            $session = \Stripe\Checkout\Session::retrieve($sessionId);
        }

        return inertia('Checkout/Success', [
            'message' => 'Payment successful! Thank you for your purchase.',
        ]);
    }

    /**
     * Handle canceled checkout.
     */
    public function cancel()
    {
        return inertia('Checkout/Cancel', [
            'message' => 'Checkout was canceled.',
        ]);
    }

    /**
     * Resolve product key to Stripe Price ID.
     */
    protected function resolvePriceId(string $productKey): string
    {
        $products = config('stripe.products');

        return $products[$productKey]
            ?? throw new \InvalidArgumentException('Invalid product.');
    }
}
```

## Step 3: Add Routes

In `routes/web.php`:

```php
use App\Http\Controllers\CheckoutController;

// Authenticated checkout
Route::middleware(['auth'])->group(function () {
    Route::post('/checkout', [CheckoutController::class, 'checkout'])
        ->name('checkout.create');
});

// Guest checkout (optional)
Route::post('/checkout/guest', [CheckoutController::class, 'guestCheckout'])
    ->name('checkout.guest');

// Checkout callbacks
Route::get('/checkout/success', [CheckoutController::class, 'success'])
    ->name('checkout.success');
Route::get('/checkout/cancel', [CheckoutController::class, 'cancel'])
    ->name('checkout.cancel');
```

## Step 4: Handle Checkout Webhook

Add to your webhook controller:

```php
/**
 * Handle checkout session completed.
 */
protected function handleCheckoutSessionCompleted(array $payload): void
{
    $session = $payload['data']['object'];

    // Only handle one-time payments (not subscriptions)
    if ($session['mode'] !== 'payment') {
        return;
    }

    Log::info('Checkout completed', [
        'session_id' => $session['id'],
        'customer' => $session['customer'],
        'amount_total' => $session['amount_total'],
        'metadata' => $session['metadata'],
    ]);

    // Fulfill the order
    $this->fulfillOrder($session);
}

/**
 * Fulfill the purchased product.
 */
protected function fulfillOrder(array $session): void
{
    $metadata = $session['metadata'];

    // Find user (authenticated or by email)
    $user = null;
    if (isset($metadata['user_id'])) {
        $user = \App\Models\User::find($metadata['user_id']);
    } elseif ($session['customer_email']) {
        $user = \App\Models\User::where('email', $session['customer_email'])->first();
    }

    // Record the purchase
    \App\Models\Purchase::create([
        'user_id' => $user?->id,
        'stripe_session_id' => $session['id'],
        'stripe_payment_intent' => $session['payment_intent'],
        'product' => $metadata['product'] ?? 'unknown',
        'amount' => $session['amount_total'],
        'currency' => $session['currency'],
        'status' => 'completed',
    ]);

    // Grant access to the product
    if ($user && isset($metadata['product'])) {
        // Example: Grant permission, unlock feature, send download link
        // $user->givePermissionTo('access-' . $metadata['product']);
    }

    // Send confirmation email
    if ($session['customer_email']) {
        // Mail::to($session['customer_email'])->send(new PurchaseConfirmation($session));
    }
}
```

## Step 5: Create Purchase Model (Optional)

```bash
php artisan make:model Purchase -m
```

Migration:
```php
public function up(): void
{
    Schema::create('purchases', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->nullable()->constrained()->nullOnDelete();
        $table->string('stripe_session_id')->unique();
        $table->string('stripe_payment_intent')->nullable();
        $table->string('product');
        $table->integer('amount');
        $table->string('currency', 3)->default('usd');
        $table->string('status')->default('pending');
        $table->timestamps();
    });
}
```

## Step 6: Create Shop/Product Page

```tsx
import { router } from '@inertiajs/react';

interface Product {
    id: string;
    name: string;
    description: string;
    price: number;
}

const products: Product[] = [
    { id: 'product_a', name: 'Digital Download', description: 'Instant access', price: 19.99 },
    { id: 'product_b', name: 'Premium Template', description: 'Full source code', price: 49.99 },
];

export default function Shop() {
    const handlePurchase = (productId: string) => {
        router.post('/checkout', { product: productId });
    };

    return (
        <div className="grid md:grid-cols-2 gap-6">
            {products.map((product) => (
                <div key={product.id} className="border rounded-lg p-6">
                    <h3 className="text-xl font-bold">{product.name}</h3>
                    <p className="text-gray-600">{product.description}</p>
                    <p className="text-2xl font-bold mt-4">${product.price}</p>
                    <button
                        onClick={() => handlePurchase(product.id)}
                        className="mt-4 bg-green-600 text-white px-6 py-2 rounded"
                    >
                        Buy Now
                    </button>
                </div>
            ))}
        </div>
    );
}
```

## Step 7: Test One-Time Payments

```bash
# Start server
php artisan serve

# Forward webhooks
stripe listen --forward-to localhost:8000/stripe/webhook

# Use test card: 4242 4242 4242 4242
```

</process>

<success_criteria>
One-time payments are working when:
- [ ] Products configured in Stripe Dashboard
- [ ] Clicking "Buy" redirects to Stripe Checkout
- [ ] Successful payment redirects to success page
- [ ] Webhook `checkout.session.completed` fires
- [ ] Purchase recorded in database
- [ ] Product access granted after purchase
- [ ] Guest checkout works (optional)
</success_criteria>

<anti_patterns>
Avoid:
- Fulfilling orders on success URL (use webhooks)
- Not recording purchases (need audit trail)
- Missing metadata (can't identify what was purchased)
- Skipping signature verification on webhooks
</anti_patterns>
