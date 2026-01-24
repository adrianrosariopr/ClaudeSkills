<overview>
Stripe Customer Portal is a hosted page where customers can manage their billing: update payment methods, view invoices, change plans, and cancel subscriptions. It's fully hosted by Stripe, reducing your development and PCI compliance burden.
</overview>

<portal_features>
| Feature | Description | Configurable |
|---------|-------------|--------------|
| Invoice history | View and download past invoices | Yes |
| Payment methods | Add, remove, set default card | Yes |
| Subscription management | View current subscription | Always on |
| Cancel subscription | Cancel at period end | Yes |
| Plan switching | Change to different plan | Yes |
| Customer info | Update billing details | Yes |
</portal_features>

<configuration>
**Stripe Dashboard:**
1. Go to Settings → Billing → Customer portal
2. Or direct: https://dashboard.stripe.com/settings/billing/portal

**Configure features:**
```
✓ Invoice history - Let customers view invoices
✓ Payment methods - Let customers update cards
✓ Subscriptions - Let customers cancel
✓ Subscription switching - Let customers change plans (add products)
```

**Cancellation options:**
- Allow cancellation: Yes/No
- Cancellation reason: Optional (collects feedback)
- Proration: How to handle mid-cycle changes

**Products for plan switching:**
- Add Basic and Pro products
- Enables upgrade/downgrade in portal
</configuration>

<redirecting_to_portal>
**Via Laravel Cashier:**
```php
// Redirect immediately
return $user->redirectToBillingPortal(route('billing'));

// Get URL only
$url = $user->billingPortalUrl(route('billing'));
```

**Controller:**
```php
public function redirectToPortal(Request $request)
{
    return $request->user()->redirectToBillingPortal(
        route('billing.index')  // Return URL after portal
    );
}
```

**Route:**
```php
Route::post('/billing/portal', [BillingController::class, 'redirectToPortal'])
    ->name('billing.portal');
```

**Button:**
```jsx
<button onClick={() => router.post('/billing/portal')}>
    Manage Billing
</button>
```
</redirecting_to_portal>

<portal_sessions>
**Create session with specific options:**
```php
$session = \Stripe\BillingPortal\Session::create([
    'customer' => $user->stripe_id,
    'return_url' => route('billing.index'),

    // Optional: Customize what customer can do
    'configuration' => $configurationId,

    // Optional: Start on specific flow
    'flow_data' => [
        'type' => 'subscription_cancel',
        'subscription_cancel' => [
            'subscription' => $subscriptionId,
        ],
    ],
]);

return redirect($session->url);
```

**Flow types:**
- `payment_method_update` - Update payment method
- `subscription_cancel` - Cancel subscription
- `subscription_update` - Change plan
- `subscription_update_confirm` - Confirm plan change
</portal_sessions>

<portal_configurations>
Create multiple configurations for different scenarios:

```php
// Via API
$config = \Stripe\BillingPortal\Configuration::create([
    'business_profile' => [
        'headline' => 'Manage your subscription',
    ],
    'features' => [
        'customer_update' => [
            'allowed_updates' => ['email', 'address'],
            'enabled' => true,
        ],
        'invoice_history' => ['enabled' => true],
        'payment_method_update' => ['enabled' => true],
        'subscription_cancel' => [
            'enabled' => true,
            'mode' => 'at_period_end',
            'cancellation_reason' => [
                'enabled' => true,
                'options' => ['too_expensive', 'missing_features', 'other'],
            ],
        ],
        'subscription_update' => [
            'enabled' => true,
            'default_allowed_updates' => ['price'],
            'products' => [
                [
                    'product' => 'prod_basic',
                    'prices' => ['price_basic_monthly', 'price_basic_yearly'],
                ],
                [
                    'product' => 'prod_pro',
                    'prices' => ['price_pro_monthly', 'price_pro_yearly'],
                ],
            ],
        ],
    ],
]);
```
</portal_configurations>

<branding>
**Dashboard → Settings → Branding:**
- Logo
- Icon
- Brand color
- Accent color
- Business name

These apply to:
- Customer Portal
- Checkout
- Invoices
- Emails
</branding>

<webhook_events>
Portal actions trigger standard webhooks:

| Portal Action | Webhook Event |
|---------------|---------------|
| Cancel subscription | `customer.subscription.updated` (cancel_at_period_end = true) |
| Change plan | `customer.subscription.updated` |
| Update payment | `payment_method.attached`, `customer.updated` |

Handle these normally in your webhook controller.
</webhook_events>

<building_custom_billing_page>
Complement the portal with your own billing page:

```php
public function billingIndex(Request $request)
{
    $user = $request->user();
    $subscription = $user->subscription('default');

    return inertia('Billing/Index', [
        'subscription' => $subscription ? [
            'name' => $subscription->name,
            'status' => $subscription->stripe_status,
            'ends_at' => $subscription->ends_at?->format('F j, Y'),
            'on_trial' => $subscription->onTrial(),
            'on_grace_period' => $subscription->onGracePeriod(),
        ] : null,

        'invoices' => $user->invoices()->map(fn ($invoice) => [
            'id' => $invoice->id,
            'date' => $invoice->date()->format('F j, Y'),
            'total' => $invoice->total(),
        ]),

        'payment_method' => $user->defaultPaymentMethod() ? [
            'brand' => $user->defaultPaymentMethod()->card->brand,
            'last4' => $user->defaultPaymentMethod()->card->last4,
        ] : null,
    ]);
}
```

**Page shows:**
- Current plan and status
- Payment method (masked)
- Invoice history with download links
- "Manage Billing" button → Portal
</building_custom_billing_page>

<invoice_downloads>
```php
// Download route
Route::get('/billing/invoice/{invoiceId}', function (Request $request, $invoiceId) {
    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => config('app.name'),
        'product' => 'Subscription',
    ]);
})->name('billing.invoice.download');
```

**Link in frontend:**
```jsx
<a href={`/billing/invoice/${invoice.id}`}>Download</a>
```
</invoice_downloads>

<when_to_use_portal_vs_custom>
**Use Stripe Portal for:**
- Payment method updates (PCI compliance)
- Invoice viewing/downloading
- Subscription cancellation
- Plan switching

**Build custom for:**
- Subscription status display
- Custom upgrade/downgrade flows
- Usage-based billing displays
- Multi-subscription management
- Custom cancellation surveys
</when_to_use_portal_vs_custom>

<common_issues>
| Issue | Cause | Fix |
|-------|-------|-----|
| Portal doesn't show products | Products not added | Add products in portal config |
| Can't switch plans | Subscription update disabled | Enable in portal settings |
| Wrong return URL | Not specified | Pass return_url parameter |
| Portal doesn't load | Customer doesn't exist in Stripe | Create customer first |
</common_issues>
