# Workflow: Test Stripe Integration

<required_reading>
**Read these reference files NOW:**
1. references/testing.md
</required_reading>

<process>

## Step 1: Install Stripe CLI

```bash
# macOS
brew install stripe/stripe-cli/stripe

# Linux
curl -s https://packages.stripe.dev/api/security/keypair/stripe-cli-gpg/public | gpg --dearmor | sudo tee /usr/share/keyrings/stripe.gpg
echo "deb [signed-by=/usr/share/keyrings/stripe.gpg] https://packages.stripe.dev/stripe-cli-debian-local stable main" | sudo tee -a /etc/apt/sources.list.d/stripe.list
sudo apt update && sudo apt install stripe

# Verify installation
stripe --version
```

## Step 2: Authenticate Stripe CLI

```bash
stripe login
# Opens browser for authentication
# Grants CLI access to your Stripe account
```

## Step 3: Forward Webhooks to Local

```bash
stripe listen --forward-to localhost:8000/stripe/webhook
```

**Important:** Copy the webhook signing secret displayed (`whsec_...`) to your `.env`:
```env
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
```

Keep this terminal running during development.

## Step 4: Test Cards Reference

| Scenario | Card Number | CVC | Expiry |
|----------|-------------|-----|--------|
| **Success** | 4242 4242 4242 4242 | Any | Any future |
| **Requires auth (3DS)** | 4000 0025 0000 3155 | Any | Any future |
| **Declined** | 4000 0000 0000 0002 | Any | Any future |
| **Insufficient funds** | 4000 0000 0000 9995 | Any | Any future |
| **Expired card** | 4000 0000 0000 0069 | Any | Any future |
| **Processing error** | 4000 0000 0000 0119 | Any | Any future |

**For all test cards:** Use any future expiry date, any 3-digit CVC, any postal code.

## Step 5: Test Subscription Flow

1. **Start local server:**
```bash
php artisan serve
npm run dev  # If using Vite
```

2. **Test new subscription:**
- Go to `/pricing`
- Select a plan
- Use test card `4242 4242 4242 4242`
- Complete checkout
- Verify redirect to success page
- Check webhook received in CLI terminal
- Verify database: `php artisan tinker` → `User::find(1)->subscribed('default')`

3. **Test subscription upgrade:**
```php
// In tinker or test
$user->subscription('default')->swap(config('stripe.prices.pro_monthly'));
```

4. **Test subscription cancellation:**
```php
$user->subscription('default')->cancel();
// Check: $user->subscription('default')->onGracePeriod() === true
```

5. **Test subscription resume:**
```php
$user->subscription('default')->resume();
```

## Step 6: Trigger Test Webhook Events

```bash
# Subscription events
stripe trigger customer.subscription.created
stripe trigger customer.subscription.updated
stripe trigger customer.subscription.deleted

# Invoice events
stripe trigger invoice.payment_succeeded
stripe trigger invoice.payment_failed
stripe trigger invoice.upcoming

# Charge events
stripe trigger charge.succeeded
stripe trigger charge.failed
stripe trigger charge.refunded

# Checkout events
stripe trigger checkout.session.completed
```

Watch your Laravel logs:
```bash
tail -f storage/logs/laravel.log
```

## Step 7: Test 3D Secure / Authentication

Use card `4000 0025 0000 3155` to test 3D Secure flow:
1. Enter card in checkout
2. Stripe shows authentication modal
3. Click "Complete authentication"
4. Verify payment completes

## Step 8: Test Failed Payments

Use card `4000 0000 0000 0002` (declined):
1. Attempt checkout
2. Verify error shown
3. Verify webhook `invoice.payment_failed` received
4. Check your error handling

## Step 9: Test Customer Portal

1. Create a subscription
2. Visit `/billing`
3. Click "Manage Billing"
4. Verify redirect to Stripe portal
5. Test updating payment method
6. Test canceling subscription

## Step 10: Write Automated Tests

Create `tests/Feature/SubscriptionTest.php`:

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Laravel\Cashier\Cashier;
use Tests\TestCase;

class SubscriptionTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();

        // Skip if no Stripe keys configured
        if (! config('cashier.secret')) {
            $this->markTestSkipped('Stripe not configured');
        }
    }

    public function test_user_can_create_stripe_customer(): void
    {
        $user = User::factory()->create();

        $user->createAsStripeCustomer();

        $this->assertNotNull($user->stripe_id);
        $this->assertStringStartsWith('cus_', $user->stripe_id);
    }

    public function test_user_can_check_subscription_status(): void
    {
        $user = User::factory()->create();

        $this->assertFalse($user->subscribed('default'));
    }

    public function test_checkout_requires_authentication(): void
    {
        $response = $this->post('/subscription/checkout', [
            'price' => 'basic_monthly',
        ]);

        $response->assertRedirect('/login');
    }

    public function test_checkout_validates_price(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/subscription/checkout', [
            'price' => 'invalid_price',
        ]);

        $response->assertSessionHasErrors('price');
    }
}
```

Run tests:
```bash
php artisan test --filter=SubscriptionTest
```

## Step 11: Test in Stripe Test Mode Dashboard

1. Go to [Stripe Dashboard](https://dashboard.stripe.com)
2. Ensure "Test mode" is ON (toggle in sidebar)
3. Check **Customers** - verify test customers created
4. Check **Subscriptions** - verify test subscriptions
5. Check **Payments** - verify test payments
6. Check **Developers** → **Events** - verify events logged
7. Check **Developers** → **Webhooks** - verify deliveries

</process>

<success_criteria>
Testing is complete when:
- [ ] Stripe CLI installed and authenticated
- [ ] Webhooks forwarding to local
- [ ] Successful subscription checkout tested
- [ ] 3D Secure flow tested
- [ ] Failed payment handling tested
- [ ] Customer portal tested
- [ ] Webhook events trigger correctly
- [ ] Automated tests pass
</success_criteria>

<test_scenarios_checklist>

### Subscription Scenarios
- [ ] New subscription (monthly)
- [ ] New subscription (yearly)
- [ ] Subscription with trial
- [ ] Upgrade plan (Basic → Pro)
- [ ] Downgrade plan (Pro → Basic)
- [ ] Cancel subscription
- [ ] Resume canceled subscription
- [ ] Subscription expires after cancel

### Payment Scenarios
- [ ] Successful payment
- [ ] 3D Secure required
- [ ] Card declined
- [ ] Insufficient funds
- [ ] Payment method update

### Webhook Scenarios
- [ ] subscription.created
- [ ] subscription.updated
- [ ] subscription.deleted
- [ ] invoice.payment_succeeded
- [ ] invoice.payment_failed

### Edge Cases
- [ ] User without Stripe customer
- [ ] Multiple subscriptions (if supported)
- [ ] Coupon/discount codes
- [ ] Refund handling

</test_scenarios_checklist>
