<overview>
Common Stripe integration issues and their solutions. Organized by symptom for quick diagnosis.
</overview>

<checkout_issues>
**"No such price: price_xxx"**

Cause: Invalid Price ID or test/live mismatch.

Fix:
1. Verify Price ID in Stripe Dashboard
2. Check `.env` has correct ID
3. Ensure using test prices with test keys
4. Clear config: `php artisan config:clear`

```bash
php artisan tinker
>>> config('stripe.prices.basic_monthly')
# Verify correct price ID
```

---

**Checkout redirect fails silently**

Cause: Success/cancel URLs invalid or relative.

Fix: Use absolute URLs with route helper:
```php
'success_url' => route('subscription.success') . '?session_id={CHECKOUT_SESSION_ID}',
'cancel_url' => route('subscription.cancel'),
```

---

**"This customer has no attached payment source"**

Cause: Trying to charge without Checkout.

Fix: Always use Checkout for new payments:
```php
return $user->newSubscription('default', $priceId)->checkout([...]);
```

---

**Checkout shows wrong currency**

Cause: Price currency doesn't match Cashier config.

Fix: Ensure `CASHIER_CURRENCY` matches your Stripe prices.
</checkout_issues>

<webhook_issues>
**404 on webhook endpoint**

Cause: Route not registered or wrong URL.

Fix:
```bash
php artisan route:list | grep webhook
# Should show: POST stripe/webhook
```

If missing, Cashier auto-registers. Check `routes/web.php` doesn't override.

---

**419 CSRF error**

Cause: CSRF protection not excluded.

Fix (Laravel 11+):
```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

---

**"Webhook signature verification failed"**

Causes:
1. Wrong webhook secret
2. Using parsed body instead of raw
3. Test vs live secret mismatch

Fix:
```bash
# Check .env
STRIPE_WEBHOOK_SECRET=whsec_xxxxx

# For local dev, use secret from:
stripe listen --forward-to localhost:8000/stripe/webhook
```

---

**Webhooks not received**

Debug steps:
1. Check Stripe Dashboard → Webhooks → Recent events
2. Verify endpoint URL is correct
3. Check firewall/hosting allows POST
4. Try Stripe CLI: `stripe listen --forward-to localhost:8000/stripe/webhook`

---

**Webhook events processed multiple times**

Cause: Missing idempotency check.

Fix:
```php
if (Payment::where('stripe_invoice_id', $invoice['id'])->exists()) {
    return; // Already processed
}
```
</webhook_issues>

<subscription_issues>
**Subscription not created after checkout**

Cause: Webhook not processing.

Debug:
1. Check webhook received: `stripe listen` output
2. Check Laravel logs: `tail -f storage/logs/laravel.log`
3. Verify `customer.subscription.created` handler
4. Check database: `subscriptions` table

---

**$user->subscribed() returns false after payment**

Causes:
1. Webhook hasn't processed yet
2. Wrong subscription name
3. Database not updated

Fix:
```php
// Check subscription name (default is 'default')
$user->subscribed('default');

// Force sync from Stripe
$user->subscription('default')?->syncStripeStatus();

// Check database directly
\DB::table('subscriptions')->where('user_id', $user->id)->get();
```

---

**Subscription status stuck on "incomplete"**

Cause: Initial payment failed or requires action.

Fix:
1. Check payment method is valid
2. Verify 3D Secure completed
3. In Dashboard, check payment intent status

---

**Grace period not working**

Cause: `ends_at` not set or checking wrong method.

Fix:
```php
$subscription->cancel(); // Sets ends_at
$subscription->onGracePeriod(); // Check grace period
$subscription->canceled(); // Check if canceled (but may still be active)
```
</subscription_issues>

<customer_issues>
**"No such customer: cus_xxx"**

Causes:
1. Customer deleted in Stripe
2. Test vs live customer mismatch
3. Wrong stripe_id in database

Fix:
```php
// Clear and recreate
$user->stripe_id = null;
$user->save();
$user->createAsStripeCustomer();
```

---

**User has no stripe_id**

Cause: Customer never created.

Fix:
```php
if (! $user->stripe_id) {
    $user->createAsStripeCustomer();
}
```

Or use Checkout (creates customer automatically).
</customer_issues>

<api_issues>
**"No API key provided"**

Cause: Missing or empty STRIPE_SECRET.

Fix:
```bash
# Check .env
STRIPE_SECRET=sk_test_xxxxx

# Clear cache
php artisan config:clear
php artisan config:cache

# Verify
php artisan tinker
>>> config('cashier.secret')
```

---

**"Invalid API Key provided"**

Causes:
1. Malformed key
2. Key revoked in Stripe
3. Test key used with live mode or vice versa

Fix:
1. Get fresh key from Stripe Dashboard
2. Verify no extra spaces in `.env`
3. Ensure test/live mode matches

---

**API rate limiting (429 errors)**

Cause: Too many requests.

Fix:
1. Cache Stripe responses where appropriate
2. Use webhooks instead of polling
3. Batch operations when possible
</api_issues>

<database_issues>
**Missing subscriptions table**

Cause: Migrations not run.

Fix:
```bash
php artisan vendor:publish --tag="cashier-migrations"
php artisan migrate
```

---

**Database doesn't match Stripe**

Cause: Webhook missed or failed.

Fix:
```php
// Sync subscription status
$user->subscription('default')->syncStripeStatus();

// Or manually check Stripe
$stripeSubscription = \Stripe\Subscription::retrieve($subscription->stripe_id);
```
</database_issues>

<billing_portal_issues>
**Portal shows "No active subscription"**

Cause: Customer portal not configured or subscription not linked.

Fix:
1. Configure portal in Stripe Dashboard
2. Verify subscription exists for customer
3. Check customer ID matches

---

**Can't switch plans in portal**

Cause: Products not added to portal configuration.

Fix:
1. Dashboard → Settings → Billing → Customer portal
2. Under Subscriptions → Products
3. Add your products
</billing_portal_issues>

<debugging_commands>
```bash
# Check config
php artisan config:clear
php artisan tinker
>>> config('cashier.key')
>>> config('cashier.secret')

# Check routes
php artisan route:list | grep -E "(stripe|billing|checkout)"

# Check database
php artisan tinker
>>> User::find(1)->stripe_id
>>> \DB::table('subscriptions')->get()

# Watch logs
tail -f storage/logs/laravel.log

# Check Stripe connection
php artisan tinker
>>> \Stripe\Customer::all(['limit' => 1])

# Forward webhooks locally
stripe listen --forward-to localhost:8000/stripe/webhook
```
</debugging_commands>

<getting_help>
**Stripe Dashboard:**
- Developers → Events: All events
- Developers → Logs: API requests
- Developers → Webhooks: Delivery status

**Stripe Support:**
- Dashboard → Help
- Include: Request ID (from error), account ID

**Laravel Cashier:**
- GitHub Issues: https://github.com/laravel/cashier-stripe/issues
- Laravel Docs: https://laravel.com/docs/billing
</getting_help>
