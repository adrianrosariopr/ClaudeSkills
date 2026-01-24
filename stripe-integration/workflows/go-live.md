# Workflow: Go Live with Stripe

<required_reading>
**Read these reference files NOW:**
1. references/security.md
2. references/troubleshooting.md
</required_reading>

<process>

## Step 1: Complete Stripe Account Activation

1. Go to [Stripe Dashboard](https://dashboard.stripe.com)
2. Click **Activate your account** or go to **Settings**
3. Complete required information:
   - Business details (name, address, industry)
   - Personal details (for identity verification)
   - Bank account for payouts
4. Wait for verification (usually instant, sometimes 24-48h)

## Step 2: Get Live API Keys

1. In Stripe Dashboard, toggle OFF **View test data** (or use mode switcher)
2. Go to **Developers** → **API keys**
3. Copy your **Live** keys:
   - Publishable key: `pk_live_...`
   - Secret key: `sk_live_...`

**Never commit live keys to version control.**

## Step 3: Create Live Products and Prices

Products and prices are separate between test and live mode.

**Option A: Copy from test mode**
1. Go to each product in test mode
2. Click **Copy to live mode**

**Option B: Create new in live mode**
1. Toggle to live mode
2. Create products and prices
3. Copy new Price IDs

## Step 4: Create Live Webhook Endpoint

1. In live mode, go to **Developers** → **Webhooks**
2. Click **Add endpoint**
3. URL: `https://yourdomain.com/stripe/webhook`
4. Select events:
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`
5. Copy the **Signing secret** (`whsec_...`)

## Step 5: Configure Production Environment

Update your production `.env`:

```env
# Switch to live keys
STRIPE_KEY=pk_live_your_live_publishable_key
STRIPE_SECRET=sk_live_your_live_secret_key
STRIPE_WEBHOOK_SECRET=whsec_your_live_webhook_secret

# Update price IDs to live mode prices
STRIPE_PRICE_BASIC_MONTHLY=price_live_xxxxx
STRIPE_PRICE_BASIC_YEARLY=price_live_xxxxx
STRIPE_PRICE_PRO_MONTHLY=price_live_xxxxx
STRIPE_PRICE_PRO_YEARLY=price_live_xxxxx
```

**Deploy:**
```bash
# Clear config cache after env changes
php artisan config:clear
php artisan config:cache
```

## Step 6: Verify Production Webhook

After deployment:

1. Make a test purchase with a real card
2. In Stripe Dashboard → **Developers** → **Webhooks**
3. Click your live endpoint
4. Check **Recent events**
5. Verify 200 response code

If webhooks fail:
- Check Laravel logs: `tail -f storage/logs/laravel.log`
- Verify CSRF exclusion is deployed
- Check webhook secret matches `.env`

## Step 7: Pre-Launch Checklist

### Security
- [ ] Live API keys in production `.env` only (not in code)
- [ ] `.env` file is in `.gitignore`
- [ ] HTTPS enabled on all pages (required for Stripe)
- [ ] Webhook signature verification enabled
- [ ] CSRF protection excludes `/stripe/*`

### Configuration
- [ ] Live API keys configured
- [ ] Live webhook endpoint created
- [ ] Live Price IDs configured
- [ ] Customer Portal configured in live mode

### Testing
- [ ] Test purchase with real card (refund after)
- [ ] Verify webhook received and processed
- [ ] Verify subscription created in database
- [ ] Test customer portal in production
- [ ] Test cancellation flow

### Monitoring
- [ ] Error tracking configured (Sentry, Bugsnag, etc.)
- [ ] Laravel logs accessible
- [ ] Stripe webhook logs monitored
- [ ] Alert for failed payments configured

### Legal/Compliance
- [ ] Privacy policy includes payment processing
- [ ] Terms of service updated
- [ ] Refund policy documented
- [ ] Customer support contact available

## Step 8: Test with Real Card

1. Create a test subscription using your own card
2. Verify payment successful in Stripe Dashboard
3. Verify subscription active in your app
4. Test customer portal
5. Cancel subscription
6. Process refund in Stripe Dashboard

**Important:** This is a real charge. Refund it after testing.

## Step 9: Monitor First Few Days

After launch:

1. **Check Stripe Dashboard daily:**
   - Payments → verify successful charges
   - Customers → verify new customers
   - Subscriptions → verify active subscriptions

2. **Check webhook delivery:**
   - Developers → Webhooks → Recent events
   - Look for failed deliveries (non-200 responses)

3. **Check Laravel logs:**
   - Any Stripe-related errors?
   - Webhook processing issues?

4. **Check customer support:**
   - Any billing complaints?
   - Payment failures?

## Step 10: Ongoing Maintenance

### Weekly
- Review failed payments in Stripe Dashboard
- Check webhook delivery success rate
- Review customer support tickets

### Monthly
- Review MRR (Monthly Recurring Revenue)
- Check churn rate
- Review refund rate
- Update Stripe SDK if needed

### As Needed
- Handle disputes/chargebacks
- Process refunds
- Update pricing/plans

</process>

<success_criteria>
Production deployment is complete when:
- [ ] Stripe account fully activated
- [ ] Live API keys configured in production
- [ ] Live webhook endpoint receiving events
- [ ] Live products/prices created
- [ ] Test purchase successful with real card
- [ ] Customer portal works in production
- [ ] Monitoring and alerting configured
- [ ] Refund issued for test purchase
</success_criteria>

<rollback_plan>
If issues occur after launch:

1. **Payment failures:**
   - Check Stripe Dashboard for error details
   - Verify API keys are correct
   - Check webhook delivery

2. **Webhook failures:**
   - Temporarily increase logging
   - Check signature secret matches
   - Verify endpoint is accessible

3. **Emergency rollback:**
   - Disable checkout (add maintenance mode to checkout routes)
   - Existing subscriptions continue working
   - Fix issue before re-enabling

```php
// Emergency: Disable new checkouts
Route::post('/subscription/checkout', function () {
    return back()->with('error', 'Checkout temporarily unavailable.');
});
```
</rollback_plan>

<post_launch_checklist>

### Day 1
- [ ] Monitor webhook delivery
- [ ] Check for any errors in logs
- [ ] Verify first real customer works

### Week 1
- [ ] Review all failed payments
- [ ] Check webhook success rate
- [ ] Address any customer complaints

### Month 1
- [ ] Review subscription metrics
- [ ] Check for churn patterns
- [ ] Optimize based on data

</post_launch_checklist>
