<overview>
Stripe provides comprehensive testing tools: test mode API keys, test cards, Stripe CLI for webhook forwarding, and test clocks for time-based testing.
</overview>

<test_mode>
**Test vs Live keys:**
| Key Prefix | Mode | Real Money? |
|------------|------|-------------|
| `pk_test_`, `sk_test_` | Test | No |
| `pk_live_`, `sk_live_` | Live | Yes |

**Always use test keys during development.**

Check your mode:
```bash
php artisan tinker
>>> config('cashier.key')
# Should start with pk_test_
```
</test_mode>

<test_cards>
**Successful payments:**
| Card Number | Scenario |
|-------------|----------|
| 4242 4242 4242 4242 | Succeeds |
| 4000 0025 0000 3155 | Requires 3DS authentication |
| 4000 0000 0000 3220 | 3DS 2 authentication required |

**Declined payments:**
| Card Number | Decline Reason |
|-------------|----------------|
| 4000 0000 0000 0002 | Generic decline |
| 4000 0000 0000 9995 | Insufficient funds |
| 4000 0000 0000 9987 | Lost card |
| 4000 0000 0000 0069 | Expired card |
| 4000 0000 0000 0127 | Incorrect CVC |
| 4000 0000 0000 0119 | Processing error |

**For all test cards:**
- Expiry: Any future date (e.g., 12/34)
- CVC: Any 3 digits (e.g., 123)
- ZIP: Any 5 digits (e.g., 12345)
</test_cards>

<stripe_cli>
**Install:**
```bash
# macOS
brew install stripe/stripe-cli/stripe

# Linux (Debian/Ubuntu)
curl -s https://packages.stripe.dev/api/security/keypair/stripe-cli-gpg/public | gpg --dearmor | sudo tee /usr/share/keyrings/stripe.gpg
echo "deb [signed-by=/usr/share/keyrings/stripe.gpg] https://packages.stripe.dev/stripe-cli-debian-local stable main" | sudo tee -a /etc/apt/sources.list.d/stripe.list
sudo apt update && sudo apt install stripe
```

**Authenticate:**
```bash
stripe login
# Opens browser for Stripe Dashboard authentication
```

**Forward webhooks:**
```bash
stripe listen --forward-to localhost:8000/stripe/webhook
```

**Trigger events:**
```bash
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
stripe trigger invoice.payment_failed
stripe trigger checkout.session.completed
stripe trigger charge.refunded
```

**List available triggers:**
```bash
stripe trigger --help
```
</stripe_cli>

<testing_subscriptions>
**Create test subscription:**
1. Start app: `php artisan serve`
2. Forward webhooks: `stripe listen --forward-to localhost:8000/stripe/webhook`
3. Go to pricing page
4. Select plan
5. Enter test card: 4242 4242 4242 4242
6. Complete checkout

**Verify in code:**
```php
php artisan tinker
>>> $user = User::find(1);
>>> $user->subscribed('default')  // true
>>> $user->subscription('default')->stripe_status  // 'active'
```

**Test plan changes:**
```php
>>> $user->subscription('default')->swap(config('stripe.prices.pro_monthly'))
>>> $user->subscription('default')->stripe_price  // new price ID
```

**Test cancellation:**
```php
>>> $user->subscription('default')->cancel()
>>> $user->subscription('default')->onGracePeriod()  // true
>>> $user->subscription('default')->ends_at  // date when access ends
```

**Test resume:**
```php
>>> $user->subscription('default')->resume()
>>> $user->subscription('default')->onGracePeriod()  // false
```
</testing_subscriptions>

<testing_3d_secure>
Use card `4000 0025 0000 3155`:

1. Enter card at checkout
2. Stripe shows authentication modal
3. Click "Complete authentication" (in test mode)
4. Verify payment completes

**Failing 3DS:**
Use card `4000 0000 0000 0341`:
- Authentication will fail
- Verify your error handling
</testing_3d_secure>

<testing_failed_payments>
**Initial failure:**
Use card `4000 0000 0000 0002` at checkout.
- Checkout shows error
- No subscription created

**Renewal failure:**
1. Create subscription with good card
2. In Stripe Dashboard, update customer's card to `4000 0000 0000 0341`
3. Trigger invoice: `stripe trigger invoice.upcoming`
4. Payment fails on next billing

**Verify handling:**
- Check `invoice.payment_failed` webhook received
- Verify user notification
- Check subscription status: `past_due`
</testing_failed_payments>

<test_clocks>
For testing time-sensitive scenarios without waiting.

**Create test clock:**
```bash
stripe test_clocks create --frozen-time "2024-01-01T00:00:00Z"
```

**Advance time:**
```bash
stripe test_clocks advance tc_xxxxx --frozen-time "2024-02-01T00:00:00Z"
```

**Use cases:**
- Test trial expiration
- Test subscription renewal
- Test dunning (failed payment retries)
- Test grace period expiration

**Note:** Test clocks are for automated testing of billing scenarios.
</test_clocks>

<automated_tests>
```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class SubscriptionTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();

        if (! config('cashier.secret')) {
            $this->markTestSkipped('Stripe not configured');
        }
    }

    public function test_user_can_be_created_as_stripe_customer(): void
    {
        $user = User::factory()->create();

        $user->createAsStripeCustomer();

        $this->assertNotNull($user->stripe_id);
        $this->assertStringStartsWith('cus_', $user->stripe_id);
    }

    public function test_unsubscribed_user_returns_false(): void
    {
        $user = User::factory()->create();

        $this->assertFalse($user->subscribed('default'));
    }

    public function test_checkout_requires_auth(): void
    {
        $response = $this->post('/subscription/checkout', [
            'price' => 'basic_monthly',
        ]);

        $response->assertRedirect('/login');
    }

    public function test_checkout_validates_price(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->post('/subscription/checkout', [
                'price' => 'invalid_price',
            ]);

        $response->assertSessionHasErrors('price');
    }

    public function test_billing_page_requires_auth(): void
    {
        $response = $this->get('/billing');

        $response->assertRedirect('/login');
    }
}
```

Run tests:
```bash
php artisan test --filter=SubscriptionTest
```
</automated_tests>

<mocking_stripe>
For unit tests that shouldn't hit Stripe API:

```php
use Mockery;
use Laravel\Cashier\Subscription;

public function test_subscription_check(): void
{
    $user = User::factory()->create();

    // Mock subscription
    $subscription = Mockery::mock(Subscription::class);
    $subscription->shouldReceive('active')->andReturn(true);
    $subscription->shouldReceive('onTrial')->andReturn(false);

    $user->setRelation('subscriptions', collect([$subscription]));

    // Test your logic
    $this->assertTrue($user->subscribed('default'));
}
```

**Note:** Use real Stripe API for integration tests, mocks for unit tests.
</mocking_stripe>

<debugging_tips>
**Watch Laravel logs:**
```bash
tail -f storage/logs/laravel.log
```

**Check Stripe CLI output:**
Webhook forwarding shows:
- Event type
- Delivery status
- Response code

**Stripe Dashboard:**
- Developers → Events: See all events
- Developers → Logs: See API requests
- Developers → Webhooks: See delivery status
</debugging_tips>
