# Workflow: Debug Stripe Issues

<required_reading>
**Read these reference files NOW:**
1. references/troubleshooting.md
2. references/webhooks.md
</required_reading>

<process>

## Step 1: Identify the Problem Category

| Symptom | Category | Go to |
|---------|----------|-------|
| Checkout not working | Checkout Issues | Step 2 |
| Webhooks not received | Webhook Issues | Step 3 |
| Signature verification failed | Webhook Security | Step 4 |
| Subscription not updating | Database Sync | Step 5 |
| Customer not found | Customer Issues | Step 6 |
| API errors | Configuration | Step 7 |

## Step 2: Debug Checkout Issues

**Symptom:** Checkout redirect fails or shows error

**Check 1: Price ID is valid**
```bash
php artisan tinker
>>> config('stripe.prices.basic_monthly')
# Should return price_xxxxx

# Verify price exists in Stripe
>>> \Stripe\Price::retrieve(config('stripe.prices.basic_monthly'))
```

**Check 2: User has Stripe customer**
```php
>>> $user = User::find(1);
>>> $user->stripe_id
# If null, customer wasn't created

# Create customer
>>> $user->createAsStripeCustomer();
```

**Check 3: Success/Cancel URLs are absolute**
```php
// Wrong
'success_url' => '/success'

// Correct
'success_url' => route('subscription.success') . '?session_id={CHECKOUT_SESSION_ID}'
```

**Check 4: Environment variables loaded**
```bash
php artisan config:clear
php artisan config:cache
php artisan tinker
>>> config('cashier.key')
>>> config('cashier.secret')
```

## Step 3: Debug Webhook Issues

**Symptom:** Webhooks not being received

**Check 1: Webhook endpoint is accessible**
```bash
# From another terminal
curl -X POST http://localhost:8000/stripe/webhook -d '{}'
# Should get 400 (bad signature), not 404
```

**Check 2: CSRF protection is disabled for webhook**
```php
// In bootstrap/app.php or VerifyCsrfToken middleware
$middleware->validateCsrfTokens(except: [
    'stripe/*',
]);
```

**Check 3: Stripe CLI is forwarding**
```bash
stripe listen --forward-to localhost:8000/stripe/webhook
# Should show: Ready! Your webhook signing secret is whsec_xxxxx
```

**Check 4: Events are registered in Stripe Dashboard**
- Go to Developers → Webhooks
- Check that required events are selected
- Check "Recent events" for delivery status

**Check 5: Laravel logs**
```bash
tail -f storage/logs/laravel.log
```

## Step 4: Debug Signature Verification

**Symptom:** "Webhook signature verification failed"

**Check 1: Correct webhook secret**
```bash
# .env should have the secret from your webhook endpoint
STRIPE_WEBHOOK_SECRET=whsec_xxxxx

# For local dev with Stripe CLI, use the secret shown by:
stripe listen --forward-to localhost:8000/stripe/webhook
```

**Check 2: No whitespace in secret**
```bash
# Check for trailing spaces
cat .env | grep STRIPE_WEBHOOK_SECRET | cat -A
# Should end with $ not space$
```

**Check 3: Using raw request body**
```php
// Correct: Use raw body
$payload = @file_get_contents('php://input');

// Wrong: Using parsed request
$payload = $request->all();  // This breaks signature
```

**Check 4: Test vs Live mismatch**
- Test mode uses different webhook secrets than live mode
- Each webhook endpoint has its own secret

## Step 5: Debug Database Sync

**Symptom:** Subscription status doesn't match Stripe

**Check 1: Webhook is updating database**
```bash
# Trigger a subscription event
stripe trigger customer.subscription.updated

# Check database
php artisan tinker
>>> \App\Models\Subscription::latest()->first()
```

**Check 2: User has correct stripe_id**
```php
>>> $user = User::find(1);
>>> $user->stripe_id
# Should match customer in Stripe Dashboard
```

**Check 3: Manual sync from Stripe**
```php
// Force sync subscription from Stripe
$user->subscription('default')->syncStripeStatus();
```

**Check 4: Check subscriptions table**
```sql
SELECT * FROM subscriptions WHERE user_id = 1;
-- Check stripe_id, stripe_status, stripe_price, ends_at
```

## Step 6: Debug Customer Issues

**Symptom:** "No such customer" or customer not found

**Check 1: User has stripe_id**
```php
>>> User::whereNull('stripe_id')->count()
# Users without Stripe customers
```

**Check 2: Customer exists in Stripe**
```php
>>> \Stripe\Customer::retrieve($user->stripe_id)
# If error, customer was deleted in Stripe
```

**Check 3: Recreate customer**
```php
// If customer was deleted in Stripe
$user->stripe_id = null;
$user->save();
$user->createAsStripeCustomer();
```

## Step 7: Debug Configuration Issues

**Symptom:** API errors, authentication failures

**Check 1: API keys are correct**
```bash
# Test key format
pk_test_xxx  # Publishable (frontend)
sk_test_xxx  # Secret (backend)

# Don't mix test and live keys!
```

**Check 2: Config is loaded**
```bash
php artisan config:clear
php artisan tinker
>>> config('cashier.key')
>>> config('cashier.secret')
```

**Check 3: Stripe API version**
```php
// Check Cashier's API version
>>> \Laravel\Cashier\Cashier::STRIPE_VERSION

// Check Stripe SDK version
>>> \Stripe\Stripe::VERSION
```

**Check 4: Network issues**
```bash
# Test Stripe API connectivity
curl https://api.stripe.com/v1/customers \
  -u sk_test_xxx:
```

## Step 8: Use Stripe Dashboard for Debugging

**Events Log:**
- Dashboard → Developers → Events
- See all events and their status

**Webhook Logs:**
- Dashboard → Developers → Webhooks → [endpoint]
- See delivery attempts and responses

**API Logs:**
- Dashboard → Developers → Logs
- See all API requests and errors

**Test Clocks (for subscription testing):**
- Dashboard → Developers → Test clocks
- Simulate time passing for subscriptions

</process>

<success_criteria>
Issue is resolved when:
- [ ] Root cause identified
- [ ] Fix applied and tested
- [ ] No errors in Laravel logs
- [ ] Webhook deliveries show 200 in Stripe Dashboard
- [ ] Database state matches Stripe state
</success_criteria>

<common_error_messages>

| Error | Cause | Fix |
|-------|-------|-----|
| "No such price" | Invalid Price ID | Check `.env` price IDs match Stripe |
| "No such customer" | Customer deleted or wrong ID | Recreate customer |
| "Webhook signature verification failed" | Wrong secret or payload modified | Use correct `whsec_` secret |
| "This customer has no attached payment source" | No payment method | Use Checkout to collect payment |
| "Cannot charge a customer that has no active card" | Same as above | Use Checkout |
| "No API key provided" | Missing STRIPE_SECRET | Add to `.env` |

</common_error_messages>
