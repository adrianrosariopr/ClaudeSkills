# Workflow: Implement Billing Portal

<required_reading>
**Read these reference files NOW:**
1. references/customer-portal.md
2. references/laravel-cashier.md
</required_reading>

<process>

## Step 1: Configure Customer Portal in Stripe Dashboard

1. Go to [Stripe Dashboard → Settings → Billing → Customer portal](https://dashboard.stripe.com/settings/billing/portal)
2. Configure features:

| Feature | Recommended Setting |
|---------|---------------------|
| **Invoice history** | Enabled |
| **Customer information** | Allow updates |
| **Payment methods** | Allow updates |
| **Cancel subscription** | Enabled |
| **Switch plans** | Enable if offering upgrades/downgrades |

3. Under **Subscriptions** → **Products**:
   - Add your Basic and Pro products
   - This enables plan switching in the portal

4. Configure cancellation options:
   - Cancellation reason: Optional (good for feedback)
   - Proration: Create prorations

5. Save changes

## Step 2: Create Billing Portal Controller

Create `app/Http/Controllers/BillingPortalController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class BillingPortalController extends Controller
{
    /**
     * Redirect to Stripe Customer Portal.
     */
    public function redirect(Request $request)
    {
        return $request->user()->redirectToBillingPortal(
            route('billing.index')
        );
    }

    /**
     * Show billing overview page.
     */
    public function index(Request $request)
    {
        $user = $request->user();
        $subscription = $user->subscription('default');

        return inertia('Billing/Index', [
            'subscription' => $subscription ? [
                'name' => $subscription->name,
                'stripe_status' => $subscription->stripe_status,
                'ends_at' => $subscription->ends_at?->format('F j, Y'),
                'on_trial' => $subscription->onTrial(),
                'on_grace_period' => $subscription->onGracePeriod(),
                'trial_ends_at' => $subscription->trial_ends_at?->format('F j, Y'),
            ] : null,
            'invoices' => $user->invoices()->map(fn ($invoice) => [
                'id' => $invoice->id,
                'date' => $invoice->date()->format('F j, Y'),
                'total' => $invoice->total(),
                'status' => $invoice->status,
            ]),
            'defaultPaymentMethod' => $user->defaultPaymentMethod() ? [
                'brand' => $user->defaultPaymentMethod()->card->brand,
                'last4' => $user->defaultPaymentMethod()->card->last4,
                'exp_month' => $user->defaultPaymentMethod()->card->exp_month,
                'exp_year' => $user->defaultPaymentMethod()->card->exp_year,
            ] : null,
        ]);
    }

    /**
     * Download invoice PDF.
     */
    public function downloadInvoice(Request $request, string $invoiceId)
    {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor' => config('app.name'),
            'product' => 'Subscription',
        ]);
    }
}
```

## Step 3: Add Billing Routes

In `routes/web.php`:

```php
use App\Http\Controllers\BillingPortalController;

Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/billing', [BillingPortalController::class, 'index'])
        ->name('billing.index');

    Route::post('/billing/portal', [BillingPortalController::class, 'redirect'])
        ->name('billing.portal');

    Route::get('/billing/invoice/{invoiceId}', [BillingPortalController::class, 'downloadInvoice'])
        ->name('billing.invoice.download');
});
```

## Step 4: Create Billing Page Component

Create `resources/js/Pages/Billing/Index.tsx`:

```tsx
import { Head, router, Link } from '@inertiajs/react';

interface Props {
    subscription: {
        name: string;
        stripe_status: string;
        ends_at: string | null;
        on_trial: boolean;
        on_grace_period: boolean;
        trial_ends_at: string | null;
    } | null;
    invoices: Array<{
        id: string;
        date: string;
        total: string;
        status: string;
    }>;
    defaultPaymentMethod: {
        brand: string;
        last4: string;
        exp_month: number;
        exp_year: number;
    } | null;
}

export default function BillingIndex({ subscription, invoices, defaultPaymentMethod }: Props) {
    const openPortal = () => {
        router.post('/billing/portal');
    };

    return (
        <>
            <Head title="Billing" />
            <div className="max-w-4xl mx-auto py-12 px-4">
                <h1 className="text-2xl font-bold mb-8">Billing & Subscription</h1>

                {/* Current Plan */}
                <section className="bg-white rounded-lg shadow p-6 mb-6">
                    <h2 className="text-lg font-semibold mb-4">Current Plan</h2>
                    {subscription ? (
                        <div>
                            <p className="text-xl font-bold capitalize">{subscription.name}</p>
                            <p className="text-gray-600">
                                Status: <span className="capitalize">{subscription.stripe_status}</span>
                            </p>
                            {subscription.on_trial && (
                                <p className="text-blue-600">
                                    Trial ends: {subscription.trial_ends_at}
                                </p>
                            )}
                            {subscription.on_grace_period && (
                                <p className="text-orange-600">
                                    Access until: {subscription.ends_at}
                                </p>
                            )}
                        </div>
                    ) : (
                        <p className="text-gray-600">No active subscription</p>
                    )}
                </section>

                {/* Payment Method */}
                <section className="bg-white rounded-lg shadow p-6 mb-6">
                    <h2 className="text-lg font-semibold mb-4">Payment Method</h2>
                    {defaultPaymentMethod ? (
                        <div className="flex items-center gap-4">
                            <span className="capitalize">{defaultPaymentMethod.brand}</span>
                            <span>•••• {defaultPaymentMethod.last4}</span>
                            <span className="text-gray-500">
                                Expires {defaultPaymentMethod.exp_month}/{defaultPaymentMethod.exp_year}
                            </span>
                        </div>
                    ) : (
                        <p className="text-gray-600">No payment method on file</p>
                    )}
                </section>

                {/* Manage Billing Button */}
                <button
                    onClick={openPortal}
                    className="bg-blue-600 text-white px-6 py-2 rounded mb-8"
                >
                    Manage Billing
                </button>

                {/* Invoice History */}
                <section className="bg-white rounded-lg shadow p-6">
                    <h2 className="text-lg font-semibold mb-4">Invoice History</h2>
                    {invoices.length > 0 ? (
                        <table className="w-full">
                            <thead>
                                <tr className="text-left border-b">
                                    <th className="py-2">Date</th>
                                    <th className="py-2">Amount</th>
                                    <th className="py-2">Status</th>
                                    <th className="py-2"></th>
                                </tr>
                            </thead>
                            <tbody>
                                {invoices.map((invoice) => (
                                    <tr key={invoice.id} className="border-b">
                                        <td className="py-3">{invoice.date}</td>
                                        <td className="py-3">{invoice.total}</td>
                                        <td className="py-3 capitalize">{invoice.status}</td>
                                        <td className="py-3">
                                            <a
                                                href={`/billing/invoice/${invoice.id}`}
                                                className="text-blue-600 hover:underline"
                                            >
                                                Download
                                            </a>
                                        </td>
                                    </tr>
                                ))}
                            </tbody>
                        </table>
                    ) : (
                        <p className="text-gray-600">No invoices yet</p>
                    )}
                </section>
            </div>
        </>
    );
}
```

## Step 5: Test Billing Portal

1. Create a test subscription
2. Visit `/billing`
3. Click "Manage Billing"
4. Verify redirect to Stripe Customer Portal
5. Test updating payment method
6. Test canceling subscription
7. Test downloading invoice

</process>

<success_criteria>
Billing portal is working when:
- [ ] Customer Portal configured in Stripe Dashboard
- [ ] Billing page shows subscription status
- [ ] Billing page shows payment method
- [ ] Billing page lists invoices with download links
- [ ] "Manage Billing" redirects to Stripe Customer Portal
- [ ] Users can update payment method in portal
- [ ] Users can cancel subscription in portal
- [ ] Invoice PDFs download correctly
</success_criteria>

<anti_patterns>
Avoid:
- Building custom payment forms (use Stripe's hosted portal)
- Storing card details in your database
- Not configuring portal products (breaks plan switching)
- Missing return URL (users get stuck after portal)
</anti_patterns>
