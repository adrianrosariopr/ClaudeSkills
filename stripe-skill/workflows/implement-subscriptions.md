# Workflow: Implement Subscriptions

<required_reading>
**Read these reference files NOW:**
1. references/subscriptions.md
2. references/stripe-checkout.md
3. references/laravel-cashier.md
</required_reading>

<process>

## Step 1: Create Subscription Controller

Create `app/Http/Controllers/SubscriptionController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class SubscriptionController extends Controller
{
    /**
     * Redirect to Stripe Checkout for subscription.
     */
    public function checkout(Request $request)
    {
        $request->validate([
            'price' => 'required|string',
        ]);

        $priceId = $this->resolvePriceId($request->price);

        return $request->user()
            ->newSubscription('default', $priceId)
            ->trialDays(config('stripe.trial_days', 0))
            ->checkout([
                'success_url' => route('subscription.success') . '?session_id={CHECKOUT_SESSION_ID}',
                'cancel_url' => route('subscription.cancel'),
            ]);
    }

    /**
     * Handle successful checkout redirect.
     */
    public function success(Request $request)
    {
        // Note: Don't rely solely on this - use webhooks for reliability
        return inertia('Billing/Success', [
            'message' => 'Subscription activated! Thank you for subscribing.',
        ]);
    }

    /**
     * Handle canceled checkout.
     */
    public function cancel()
    {
        return inertia('Billing/Cancel', [
            'message' => 'Checkout was canceled. No charges were made.',
        ]);
    }

    /**
     * Show subscription management page.
     */
    public function manage(Request $request)
    {
        $user = $request->user();

        return inertia('Billing/Manage', [
            'subscription' => $user->subscription('default'),
            'onTrial' => $user->onTrial('default'),
            'subscribed' => $user->subscribed('default'),
            'subscribedToProduct' => [
                'basic' => $user->subscribedToProduct(config('stripe.products.basic'), 'default'),
                'pro' => $user->subscribedToProduct(config('stripe.products.pro'), 'default'),
            ],
            'onGracePeriod' => $user->subscription('default')?->onGracePeriod(),
        ]);
    }

    /**
     * Swap subscription to a different plan.
     */
    public function swap(Request $request)
    {
        $request->validate([
            'price' => 'required|string',
        ]);

        $priceId = $this->resolvePriceId($request->price);

        $request->user()
            ->subscription('default')
            ->swap($priceId);

        return back()->with('success', 'Plan updated successfully.');
    }

    /**
     * Cancel subscription at period end.
     */
    public function cancel(Request $request)
    {
        $request->user()
            ->subscription('default')
            ->cancel();

        return back()->with('success', 'Subscription will be canceled at the end of the billing period.');
    }

    /**
     * Resume a canceled subscription (during grace period).
     */
    public function resume(Request $request)
    {
        $request->user()
            ->subscription('default')
            ->resume();

        return back()->with('success', 'Subscription resumed.');
    }

    /**
     * Resolve price key to Stripe Price ID.
     */
    protected function resolvePriceId(string $priceKey): string
    {
        $prices = [
            'basic_monthly' => config('stripe.prices.basic_monthly'),
            'basic_yearly' => config('stripe.prices.basic_yearly'),
            'pro_monthly' => config('stripe.prices.pro_monthly'),
            'pro_yearly' => config('stripe.prices.pro_yearly'),
        ];

        return $prices[$priceKey] ?? throw new \InvalidArgumentException('Invalid price key.');
    }
}
```

## Step 2: Add Routes

In `routes/web.php`:

```php
use App\Http\Controllers\SubscriptionController;

Route::middleware(['auth', 'verified'])->group(function () {
    // Subscription checkout
    Route::post('/subscription/checkout', [SubscriptionController::class, 'checkout'])
        ->name('subscription.checkout');

    // Checkout callbacks
    Route::get('/subscription/success', [SubscriptionController::class, 'success'])
        ->name('subscription.success');
    Route::get('/subscription/cancel', [SubscriptionController::class, 'cancel'])
        ->name('subscription.cancel');

    // Subscription management (requires active subscription)
    Route::middleware('subscribed')->group(function () {
        Route::get('/billing', [SubscriptionController::class, 'manage'])
            ->name('billing.manage');
        Route::post('/subscription/swap', [SubscriptionController::class, 'swap'])
            ->name('subscription.swap');
        Route::post('/subscription/cancel', [SubscriptionController::class, 'cancelSubscription'])
            ->name('subscription.cancel');
        Route::post('/subscription/resume', [SubscriptionController::class, 'resume'])
            ->name('subscription.resume');
    });
});
```

## Step 3: Create Subscription Middleware

Create `app/Http/Middleware/EnsureUserIsSubscribed.php`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class EnsureUserIsSubscribed
{
    public function handle(Request $request, Closure $next, ?string $plan = null)
    {
        if (! $request->user()?->subscribed('default')) {
            return redirect()->route('pricing');
        }

        // Optionally check for specific plan
        if ($plan && ! $request->user()->subscribedToPrice($plan, 'default')) {
            return redirect()->route('pricing')->with('error', 'Please upgrade your plan.');
        }

        return $next($request);
    }
}
```

Register in `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => \App\Http\Middleware\EnsureUserIsSubscribed::class,
    ]);
})
```

## Step 4: Create Pricing Page Component

Create `resources/js/Pages/Pricing.tsx`:

```tsx
import { Head, router } from '@inertiajs/react';

interface Plan {
    name: string;
    priceKey: string;
    monthly: number;
    yearly: number;
    features: string[];
    popular?: boolean;
}

const plans: Plan[] = [
    {
        name: 'Basic',
        priceKey: 'basic',
        monthly: 9.99,
        yearly: 99,
        features: ['Feature A', 'Feature B', 'Standard support'],
    },
    {
        name: 'Pro',
        priceKey: 'pro',
        monthly: 29.99,
        yearly: 299,
        features: ['Everything in Basic', 'Feature C', 'Feature D', 'Priority support'],
        popular: true,
    },
];

export default function Pricing({ auth }: { auth: { user: any } }) {
    const [billing, setBilling] = useState<'monthly' | 'yearly'>('monthly');

    const handleSubscribe = (priceKey: string) => {
        router.post('/subscription/checkout', {
            price: `${priceKey}_${billing}`,
        });
    };

    return (
        <>
            <Head title="Pricing" />
            <div className="max-w-5xl mx-auto py-12 px-4">
                <h1 className="text-3xl font-bold text-center mb-8">Choose Your Plan</h1>

                {/* Billing toggle */}
                <div className="flex justify-center mb-8">
                    <button
                        onClick={() => setBilling('monthly')}
                        className={billing === 'monthly' ? 'font-bold' : ''}
                    >
                        Monthly
                    </button>
                    <button
                        onClick={() => setBilling('yearly')}
                        className={billing === 'yearly' ? 'font-bold' : ''}
                    >
                        Yearly (2 months free)
                    </button>
                </div>

                {/* Plan cards */}
                <div className="grid md:grid-cols-2 gap-8">
                    {plans.map((plan) => (
                        <div key={plan.name} className="border rounded-lg p-6">
                            {plan.popular && (
                                <span className="bg-blue-500 text-white px-2 py-1 rounded text-sm">
                                    Popular
                                </span>
                            )}
                            <h2 className="text-2xl font-bold">{plan.name}</h2>
                            <p className="text-3xl font-bold mt-4">
                                ${billing === 'monthly' ? plan.monthly : plan.yearly}
                                <span className="text-sm font-normal">
                                    /{billing === 'monthly' ? 'mo' : 'yr'}
                                </span>
                            </p>
                            <ul className="mt-4 space-y-2">
                                {plan.features.map((feature) => (
                                    <li key={feature}>✓ {feature}</li>
                                ))}
                            </ul>
                            <button
                                onClick={() => handleSubscribe(plan.priceKey)}
                                className="mt-6 w-full bg-blue-600 text-white py-2 rounded"
                            >
                                Subscribe
                            </button>
                        </div>
                    ))}
                </div>
            </div>
        </>
    );
}
```

## Step 5: Test Subscription Flow

```bash
# Start local server
php artisan serve

# In another terminal, forward webhooks
stripe listen --forward-to localhost:8000/stripe/webhook

# Use test card: 4242 4242 4242 4242
```

</process>

<success_criteria>
Subscriptions are working when:
- [ ] Pricing page shows plans with monthly/yearly toggle
- [ ] Clicking subscribe redirects to Stripe Checkout
- [ ] Successful payment redirects to success page
- [ ] Webhook updates subscription in database
- [ ] User can view subscription status
- [ ] Plan swap works with proration
- [ ] Cancel sets `ends_at` (grace period)
- [ ] Resume works during grace period
</success_criteria>

<anti_patterns>
Avoid:
- Relying on success URL to activate subscription (use webhooks)
- Hardcoding price IDs in frontend
- Forgetting to handle grace period state
- Not validating price keys before checkout
</anti_patterns>
