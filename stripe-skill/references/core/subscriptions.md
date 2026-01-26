<overview>
Stripe subscriptions enable recurring billing. Laravel Cashier provides a fluent interface for managing the subscription lifecycle: creation, modification, cancellation, and resumption.
</overview>

<subscription_lifecycle>
```
Customer selects plan
        │
        ▼
Checkout Session created → Customer redirected to Stripe
        │
        ▼
Customer completes payment
        │
        ├──▶ Webhook: checkout.session.completed
        │
        ▼
Subscription created in Stripe
        │
        ├──▶ Webhook: customer.subscription.created
        │
        ▼
Invoice created and paid
        │
        ├──▶ Webhook: invoice.payment_succeeded
        │
        ▼
Subscription active (repeats monthly/yearly)
```
</subscription_lifecycle>

<subscription_statuses>
| Status | Meaning | Access? |
|--------|---------|---------|
| `trialing` | In trial period | Yes |
| `active` | Paid and current | Yes |
| `past_due` | Payment failed, retrying | Depends on grace |
| `unpaid` | All retries failed | No |
| `canceled` | Canceled, may be in grace period | Check `ends_at` |
| `incomplete` | Initial payment failed | No |
| `incomplete_expired` | Initial payment never completed | No |
| `paused` | Subscription paused | No |

**Check status in code:**
```php
$subscription = $user->subscription('default');
$subscription->stripe_status;  // active, past_due, etc.
$subscription->active();       // Helper method
```
</subscription_statuses>

<creating_subscriptions>
**Via Checkout (recommended):**
```php
return $user->newSubscription('default', $priceId)
    ->checkout([
        'success_url' => route('success') . '?session_id={CHECKOUT_SESSION_ID}',
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

**Allow promotion codes:**
```php
return $user->newSubscription('default', $priceId)
    ->allowPromotionCodes()
    ->checkout([...]);
```
</creating_subscriptions>

<checking_subscription_status>
```php
// Is user subscribed?
$user->subscribed('default');              // Any active subscription?
$user->subscribedToProduct($productId);     // Specific product?
$user->subscribedToPrice($priceId);        // Specific price?

// Trial status
$user->onTrial('default');                 // Currently on trial?
$user->hasExpiredTrial('default');         // Trial expired?

// Grace period (canceled but access until period end)
$subscription->onGracePeriod();            // In grace period?
$subscription->canceled();                 // Has been canceled?
$subscription->ended();                    // Fully ended?

// Generic trial (not tied to subscription)
$user->onGenericTrial();
```
</checking_subscription_status>

<modifying_subscriptions>
**Swap plans (with proration):**
```php
$user->subscription('default')->swap($newPriceId);
```

**Swap without proration:**
```php
$user->subscription('default')
    ->noProrate()
    ->swap($newPriceId);
```

**Swap and invoice immediately:**
```php
$user->subscription('default')->swapAndInvoice($newPriceId);
```

**Update quantity:**
```php
$user->subscription('default')->updateQuantity(5);
$user->subscription('default')->incrementQuantity();
$user->subscription('default')->decrementQuantity();
```
</modifying_subscriptions>

<canceling_subscriptions>
**Cancel at period end (grace period):**
```php
$user->subscription('default')->cancel();
// User keeps access until current period ends
// $subscription->onGracePeriod() returns true
```

**Cancel immediately:**
```php
$user->subscription('default')->cancelNow();
// Access revoked immediately
// Prorated refund depends on Stripe settings
```

**Cancel and refund:**
```php
$user->subscription('default')->cancelNowAndInvoice();
// Creates final invoice with prorated credit
```
</canceling_subscriptions>

<resuming_subscriptions>
During grace period only:

```php
if ($user->subscription('default')->onGracePeriod()) {
    $user->subscription('default')->resume();
}
```

This:
- Clears `ends_at` date
- Continues billing cycle where it left off
</resuming_subscriptions>

<proration>
**What is proration?**
When a customer changes plans mid-cycle, Stripe calculates the difference.

**Upgrade:** Customer pays the difference immediately
**Downgrade:** Credit applied to next invoice

**Control proration behavior:**
```php
// Default: Always prorate
$subscription->swap($newPrice);

// No proration
$subscription->noProrate()->swap($newPrice);

// Prorate and invoice immediately
$subscription->swapAndInvoice($newPrice);
```
</proration>

<trials>
**Subscription trial:**
```php
$user->newSubscription('default', $priceId)
    ->trialDays(14)
    ->checkout([...]);
```

**Generic trial (not tied to subscription):**
```php
$user->createAsStripeCustomer([
    'trial_ends_at' => now()->addDays(14),
]);

// Check
$user->onGenericTrial();  // true during trial
```

**Skip trial for returning customer:**
```php
$user->newSubscription('default', $priceId)
    ->skipTrial()
    ->checkout([...]);
```
</trials>

<multiple_subscriptions>
```php
// Create with different names
$user->newSubscription('main', $mainPrice)->checkout([...]);
$user->newSubscription('addon', $addonPrice)->checkout([...]);

// Check specific subscription
$user->subscribed('main');
$user->subscribed('addon');

// Get specific subscription
$main = $user->subscription('main');
$addon = $user->subscription('addon');
```
</multiple_subscriptions>

<access_control_patterns>
**Middleware:**
```php
// Check any subscription
if (! $user->subscribed('default')) {
    return redirect('/pricing');
}

// Check specific plan
if (! $user->subscribedToPrice(config('stripe.prices.pro_monthly'))) {
    return redirect('/upgrade');
}

// Check with grace period
if ($user->subscribed('default') || $user->subscription('default')?->onGracePeriod()) {
    // Allow access during grace period
}
```

**Policy/Gate:**
```php
Gate::define('access-pro-feature', function ($user) {
    return $user->subscribedToProduct(config('stripe.products.pro'));
});
```

**Blade:**
```blade
@if(auth()->user()->subscribed('default'))
    <p>You have an active subscription!</p>
@endif
```
</access_control_patterns>

<webhook_events>
| Event | When | Action |
|-------|------|--------|
| `customer.subscription.created` | New subscription | Grant access |
| `customer.subscription.updated` | Plan changed, status changed | Update access level |
| `customer.subscription.deleted` | Subscription ended | Revoke access |
| `invoice.payment_succeeded` | Renewal paid | Log, send receipt |
| `invoice.payment_failed` | Payment failed | Notify user, start dunning |

Cashier handles most of these automatically. Override webhook controller for custom logic.
</webhook_events>

<subscription_anchors>
**Billing anchor:** The day of month billing occurs.

**Change anchor (on swap):**
```php
// Reset billing cycle to today
$subscription->anchorBillingCycleOn(now());

// Keep current anchor
$subscription->swap($newPrice);  // Default behavior
```
</subscription_anchors>
