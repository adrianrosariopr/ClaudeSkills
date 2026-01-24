<overview>
Laravel Cashier provides an expressive, fluent interface to Stripe's subscription billing services. It handles customer management, subscriptions, checkout sessions, invoices, and webhook processing.
</overview>

<installation>
```bash
composer require laravel/cashier
php artisan vendor:publish --tag="cashier-migrations"
php artisan migrate
```
</installation>

<billable_trait>
Add to your User model:

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

This adds methods:
- `createAsStripeCustomer()` - Create Stripe customer
- `checkout()` - Create checkout session
- `newSubscription()` - Start subscription
- `subscription()` - Get subscription
- `subscribed()` - Check if subscribed
- `onTrial()` - Check if on trial
- `invoices()` - Get invoices
</billable_trait>

<customer_management>
**Create customer:**
```php
$user->createAsStripeCustomer();
// Creates customer in Stripe, saves stripe_id to users table
```

**Create customer with metadata:**
```php
$user->createAsStripeCustomer([
    'name' => $user->name,
    'email' => $user->email,
    'metadata' => ['user_id' => $user->id],
]);
```

**Update Stripe customer:**
```php
$user->updateStripeCustomer([
    'name' => $user->name,
]);
```

**Get Stripe customer:**
```php
$stripeCustomer = $user->asStripeCustomer();
```
</customer_management>

<checkout_sessions>
**Subscription checkout:**
```php
return $user->newSubscription('default', $priceId)
    ->checkout([
        'success_url' => route('success') . '?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('cancel'),
    ]);
```

**One-time checkout:**
```php
return $user->checkout([
    $priceId => $quantity,
], [
    'success_url' => route('success'),
    'cancel_url' => route('cancel'),
]);
```

**With trial:**
```php
return $user->newSubscription('default', $priceId)
    ->trialDays(7)
    ->checkout([...]);
```

**With coupon:**
```php
return $user->newSubscription('default', $priceId)
    ->withCoupon('DISCOUNT20')
    ->checkout([...]);
```

**With promotion code:**
```php
return $user->newSubscription('default', $priceId)
    ->allowPromotionCodes()
    ->checkout([...]);
```
</checkout_sessions>

<subscription_methods>
**Check subscription:**
```php
$user->subscribed('default');           // Has active subscription?
$user->subscribedToProduct($product);   // Subscribed to specific product?
$user->subscribedToPrice($price);       // Subscribed to specific price?
$user->onTrial('default');              // On trial?
$user->hasExpiredTrial('default');      // Trial expired?
```

**Get subscription:**
```php
$subscription = $user->subscription('default');
$subscription->stripe_status;           // active, canceled, past_due, etc.
$subscription->stripe_price;            // Current price ID
$subscription->trial_ends_at;           // Trial end date
$subscription->ends_at;                 // When subscription ends (if canceled)
```

**Modify subscription:**
```php
// Swap to different price (with proration)
$subscription->swap($newPriceId);

// Swap without proration
$subscription->noProrate()->swap($newPriceId);

// Swap and invoice immediately
$subscription->swapAndInvoice($newPriceId);

// Cancel at period end
$subscription->cancel();

// Cancel immediately
$subscription->cancelNow();

// Resume (if on grace period)
$subscription->resume();
```

**Grace period:**
```php
$subscription->onGracePeriod();  // Canceled but still active until period ends
$subscription->ended();          // Subscription has fully ended
```
</subscription_methods>

<invoices>
**Get invoices:**
```php
$invoices = $user->invoices();          // All invoices
$invoice = $user->findInvoice($id);     // Specific invoice
$invoices = $user->invoicesIncludingPending();
```

**Invoice data:**
```php
$invoice->id;
$invoice->date();                       // Carbon instance
$invoice->total();                      // Formatted total
$invoice->rawTotal();                   // Amount in cents
$invoice->status;                       // paid, open, void, etc.
```

**Download invoice:**
```php
return $user->downloadInvoice($invoiceId, [
    'vendor' => 'Your Company',
    'product' => 'Subscription',
]);
```
</invoices>

<payment_methods>
**Default payment method:**
```php
$paymentMethod = $user->defaultPaymentMethod();
$paymentMethod->card->brand;            // visa, mastercard, etc.
$paymentMethod->card->last4;            // Last 4 digits
$paymentMethod->card->exp_month;
$paymentMethod->card->exp_year;
```

**All payment methods:**
```php
$paymentMethods = $user->paymentMethods();
```

**Update default:**
```php
$user->updateDefaultPaymentMethod($paymentMethodId);
```
</payment_methods>

<billing_portal>
**Redirect to Stripe portal:**
```php
return $user->redirectToBillingPortal(route('billing'));
```

**Get portal URL:**
```php
$url = $user->billingPortalUrl(route('billing'));
```

The portal allows customers to:
- Update payment method
- View invoice history
- Cancel subscription
- Switch plans (if configured)
</billing_portal>

<configuration>
**config/cashier.php:**
```php
return [
    'key' => env('STRIPE_KEY'),
    'secret' => env('STRIPE_SECRET'),
    'webhook' => [
        'secret' => env('STRIPE_WEBHOOK_SECRET'),
        'tolerance' => env('STRIPE_WEBHOOK_TOLERANCE', 300),
    ],
    'currency' => env('CASHIER_CURRENCY', 'usd'),
    'currency_locale' => env('CASHIER_CURRENCY_LOCALE', 'en'),
];
```

**Environment variables:**
```env
STRIPE_KEY=pk_test_xxx
STRIPE_SECRET=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
CASHIER_CURRENCY=usd
```
</configuration>

<database_tables>
**customers (added to users table):**
- `stripe_id` - Stripe customer ID (cus_xxx)
- `pm_type` - Default payment method type
- `pm_last_four` - Last 4 digits
- `trial_ends_at` - Generic trial end date

**subscriptions:**
- `user_id` - Foreign key
- `type` - Subscription name (e.g., 'default')
- `stripe_id` - Stripe subscription ID
- `stripe_status` - active, canceled, past_due, etc.
- `stripe_price` - Current price ID
- `quantity` - Item quantity
- `trial_ends_at` - Trial end
- `ends_at` - Cancellation end date

**subscription_items:**
- `subscription_id` - Foreign key
- `stripe_id` - Stripe item ID
- `stripe_product` - Product ID
- `stripe_price` - Price ID
- `quantity` - Item quantity
</database_tables>

<when_to_use_cashier_vs_stripe_directly>
**Use Cashier for:**
- Customer management
- Subscription lifecycle
- Checkout sessions
- Invoice retrieval
- Billing portal
- Webhook handling

**Use Stripe SDK directly for:**
- Custom Checkout configuration
- Payment Intents (non-Checkout)
- Complex pricing (metered, tiered)
- Stripe Connect
- Advanced webhook handling
</when_to_use_cashier_vs_stripe_directly>
