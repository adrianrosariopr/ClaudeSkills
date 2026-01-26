<pricing_overview>
Stripe pricing models, strategies, and implementation patterns. Choose the right model for your business.
</pricing_overview>

<flat_rate_subscriptions>
## Flat-Rate Subscriptions

**Best for:** Simple SaaS with consistent feature sets per tier.

**Example:** $10/month for Basic, $25/month for Pro, $50/month for Enterprise.

**Implementation:**
```php
// Create prices in Stripe Dashboard or via API
$basicPrice = Stripe::prices()->create([
    'unit_amount' => 1000, // $10.00
    'currency' => 'usd',
    'recurring' => ['interval' => 'month'],
    'product' => 'prod_basic',
]);

// config/stripe.php
'tiers' => [
    'basic' => [
        'monthly' => 'price_basic_monthly',
        'yearly' => 'price_basic_yearly',
        'features' => ['10 projects', 'Email support'],
    ],
    'pro' => [
        'monthly' => 'price_pro_monthly',
        'yearly' => 'price_pro_yearly',
        'features' => ['Unlimited projects', 'Priority support', 'API access'],
    ],
],

// Checkout
$user->newSubscription('default', config("stripe.tiers.{$tier}.monthly"))
    ->checkout();
```

**Pros:**
- Simple to understand and implement
- Predictable revenue
- Easy to communicate pricing

**Cons:**
- May leave money on the table from power users
- Harder to upsell within a tier
</flat_rate_subscriptions>

<per_seat_pricing>
## Per-Seat / Per-User Pricing

**Best for:** Team collaboration tools where value scales with users.

**Example:** $8/user/month.

**Implementation:**
```php
// Create metered price
$seatPrice = Stripe::prices()->create([
    'unit_amount' => 800, // $8.00 per seat
    'currency' => 'usd',
    'recurring' => ['interval' => 'month'],
    'product' => 'prod_team_plan',
]);

// Create subscription with quantity
$user->newSubscription('default', 'price_per_seat')
    ->quantity($team->members()->count())
    ->checkout();

// When team size changes
public function addTeamMember(Team $team)
{
    // Add member logic...

    $team->owner->subscription('default')
        ->incrementQuantity();
}

public function removeTeamMember(Team $team)
{
    // Remove member logic...

    $team->owner->subscription('default')
        ->decrementQuantity();
}

// Show current cost
$subscription = $user->subscription('default');
$quantity = $subscription->quantity;
$unitPrice = 8; // $8/seat
$monthlyTotal = $quantity * $unitPrice;
```

**Pros:**
- Revenue scales with customer growth
- Easy for customers to understand
- Natural upsell as teams grow

**Cons:**
- Customers may limit users to save money
- Need to handle seat changes (add/remove)
</per_seat_pricing>

<usage_based_pricing>
## Usage-Based / Metered Billing

**Best for:** API providers, cloud services, where usage varies significantly.

**Example:** $0.001 per API call, $0.10 per GB stored.

**Implementation:**
```php
// Create metered price
$apiPrice = Stripe::prices()->create([
    'currency' => 'usd',
    'recurring' => [
        'interval' => 'month',
        'usage_type' => 'metered',
    ],
    'unit_amount_decimal' => '0.1', // $0.001 per unit
    'product' => 'prod_api_usage',
]);

// Subscribe (no immediate charge)
$user->newSubscription('default', 'price_metered_api')
    ->meteredPrice('price_metered_api')
    ->checkout();

// Track usage throughout billing period
public function handleApiRequest(Request $request)
{
    // Process request...

    // Report usage
    $subscriptionItem = $request->user()
        ->subscription('default')
        ->items
        ->first();

    Stripe::subscriptionItems()->createUsageRecord(
        $subscriptionItem->stripe_id,
        [
            'quantity' => 1,
            'timestamp' => now()->timestamp,
            'action' => 'increment',
        ]
    );
}

// Batch reporting (more efficient)
Schedule::command('report:stripe-usage')->hourly();

public function handle()
{
    $users = User::whereHas('subscriptions')->get();

    foreach ($users as $user) {
        $usage = ApiUsage::where('user_id', $user->id)
            ->where('reported', false)
            ->count();

        if ($usage > 0) {
            $subscriptionItem = $user->subscription('default')->items->first();

            Stripe::subscriptionItems()->createUsageRecord(
                $subscriptionItem->stripe_id,
                ['quantity' => $usage, 'action' => 'increment']
            );

            ApiUsage::where('user_id', $user->id)
                ->where('reported', false)
                ->update(['reported' => true]);
        }
    }
}
```

**Pros:**
- Fair pricing aligned with value delivered
- Low barrier to entry
- Revenue scales with customer success

**Cons:**
- Unpredictable revenue
- Complex to track and report usage
- Customers may be surprised by bills
</usage_based_pricing>

<tiered_pricing>
## Tiered Volume Pricing

**Best for:** When unit cost should decrease at higher volumes.

**Example:**
- 0-100 API calls: $0.02 each
- 101-1000 API calls: $0.01 each
- 1001+ API calls: $0.005 each

**Implementation:**
```php
// Create graduated pricing
$price = Stripe::prices()->create([
    'currency' => 'usd',
    'recurring' => ['interval' => 'month', 'usage_type' => 'metered'],
    'billing_scheme' => 'tiered',
    'tiers_mode' => 'graduated', // or 'volume'
    'tiers' => [
        ['up_to' => 100, 'unit_amount' => 2],      // $0.02
        ['up_to' => 1000, 'unit_amount' => 1],     // $0.01
        ['up_to' => 'inf', 'unit_amount_decimal' => '0.5'], // $0.005
    ],
    'product' => 'prod_api_calls',
]);

// graduated: Each tier priced separately
// 150 calls = (100 × $0.02) + (50 × $0.01) = $2.50

// volume: All units at single tier rate
// 150 calls = 150 × $0.01 = $1.50
```

**Show estimated cost:**
```php
public function estimateCost(int $usage): int
{
    $tiers = [
        ['up_to' => 100, 'rate' => 2],
        ['up_to' => 1000, 'rate' => 1],
        ['up_to' => PHP_INT_MAX, 'rate' => 0.5],
    ];

    $cost = 0;
    $remaining = $usage;
    $previousLimit = 0;

    foreach ($tiers as $tier) {
        $tierUsage = min($remaining, $tier['up_to'] - $previousLimit);
        $cost += $tierUsage * $tier['rate'];
        $remaining -= $tierUsage;
        $previousLimit = $tier['up_to'];

        if ($remaining <= 0) break;
    }

    return (int) $cost;
}
```
</tiered_pricing>

<hybrid_pricing>
## Hybrid: Base + Usage

**Best for:** Combining predictable base revenue with usage upside.

**Example:** $50/month base + $0.01 per API call.

**Implementation:**
```php
// Create two prices
$basePrice = Stripe::prices()->create([
    'unit_amount' => 5000, // $50 base
    'currency' => 'usd',
    'recurring' => ['interval' => 'month'],
    'product' => 'prod_api_plan',
]);

$usagePrice = Stripe::prices()->create([
    'currency' => 'usd',
    'recurring' => ['interval' => 'month', 'usage_type' => 'metered'],
    'unit_amount' => 1, // $0.01 per call
    'product' => 'prod_api_usage',
]);

// Subscribe to both
$user->newSubscription('default', 'price_base')
    ->price('price_usage') // Add second price
    ->meteredPrice('price_usage')
    ->checkout();
```

**Pros:**
- Predictable base revenue
- Captures value from power users
- Incentivizes product usage

**Cons:**
- More complex billing
- Two prices to manage
</hybrid_pricing>

<freemium>
## Freemium Model

**Best for:** Products with network effects or viral growth potential.

**Implementation:**
```php
// No Stripe involvement for free tier
public function register(Request $request)
{
    $user = User::create($request->validated());
    // No subscription created - they're on free tier
    return redirect('/dashboard');
}

// Feature gating
public function checkFeatureAccess(User $user, string $feature): bool
{
    $freeTierFeatures = ['basic_analytics', 'single_project'];
    $paidFeatures = ['advanced_analytics', 'unlimited_projects', 'api'];

    if (in_array($feature, $freeTierFeatures)) {
        return true;
    }

    return $user->subscribed('default');
}

// Upgrade prompt
@unless($user->subscribed('default'))
    <div class="upgrade-prompt">
        <p>Upgrade to Pro for unlimited projects</p>
        <a href="{{ route('billing.checkout') }}">Upgrade Now</a>
    </div>
@endunless
```

**Conversion optimization:**
```php
// Track usage approaching limits
public function createProject(Request $request)
{
    $user = $request->user();
    $count = $user->projects()->count();
    $limit = $user->subscribed() ? PHP_INT_MAX : 3;

    if ($count >= $limit) {
        return redirect()->route('billing.upgrade');
    }

    if ($count === $limit - 1 && !$user->subscribed()) {
        // One project left - show upgrade prompt
        session()->flash('upgrade_prompt', true);
    }

    // Create project...
}
```
</freemium>

<annual_discount>
## Annual Billing Discounts

**Best for:** Improving cash flow and reducing churn.

**Typical discount:** 10-20% off monthly rate.

**Implementation:**
```php
// config/stripe.php
'plans' => [
    'pro' => [
        'monthly' => [
            'price_id' => 'price_pro_monthly',
            'amount' => 29, // $29/month
        ],
        'yearly' => [
            'price_id' => 'price_pro_yearly',
            'amount' => 290, // $290/year = $24.17/month (17% savings)
        ],
    ],
],

// Show savings
$monthlyCost = 29 * 12; // $348/year
$yearlyCost = 290;
$savings = $monthlyCost - $yearlyCost; // $58
$savingsPercent = round(($savings / $monthlyCost) * 100); // 17%

// UI
<div class="pricing-toggle">
    <button @click="interval = 'monthly'">Monthly</button>
    <button @click="interval = 'yearly'">
        Yearly <span class="badge">Save {{ $savingsPercent }}%</span>
    </button>
</div>

<div class="price">
    @if($interval === 'yearly')
        <span class="amount">${{ $yearlyPrice / 12 }}/mo</span>
        <span class="billing-note">Billed annually ({{ $savings }} savings)</span>
    @else
        <span class="amount">${{ $monthlyPrice }}/mo</span>
    @endif
</div>
```

**Benefits calculation:**
- Lower churn (committed for year)
- Better cash flow (payment upfront)
- Lower transaction fees (1 transaction vs 12)
</annual_discount>

<prepaid_credits>
## Prepaid Credits / Packages

**Best for:** API services, one-time purchases with usage component.

**Example:** Buy 10,000 API credits for $50.

**Implementation:**
```php
// Credit packages
$packages = [
    ['credits' => 1000, 'price' => 10, 'bonus' => 0],
    ['credits' => 5000, 'price' => 40, 'bonus' => 500],   // 10% bonus
    ['credits' => 10000, 'price' => 70, 'bonus' => 2000], // 20% bonus
];

// Purchase credits
public function purchaseCredits(Request $request)
{
    $package = $packages[$request->package_id];

    $session = Stripe::checkout->sessions->create([
        'line_items' => [[
            'price_data' => [
                'currency' => 'usd',
                'product_data' => [
                    'name' => "{$package['credits']} API Credits",
                ],
                'unit_amount' => $package['price'] * 100,
            ],
            'quantity' => 1,
        ]],
        'mode' => 'payment',
        'metadata' => [
            'credits' => $package['credits'] + $package['bonus'],
            'user_id' => $request->user()->id,
        ],
        'success_url' => route('credits.success'),
        'cancel_url' => route('credits.purchase'),
    ]);

    return redirect($session->url);
}

// Webhook: Add credits
public function handleCheckoutCompleted($session)
{
    $user = User::find($session->metadata->user_id);
    $credits = (int) $session->metadata->credits;

    $user->increment('api_credits', $credits);
}

// Deduct on use
public function handleApiRequest(Request $request)
{
    $user = $request->user();

    if ($user->api_credits <= 0) {
        return response()->json(['error' => 'Insufficient credits'], 402);
    }

    // Process request...
    $user->decrement('api_credits');
}
```
</prepaid_credits>

<grandfathering>
## Grandfathering / Legacy Pricing

**Best for:** Keeping existing customers on old pricing during price increases.

**Implementation:**
```php
// When raising prices, create new price IDs
$newPrice = Stripe::prices()->create([
    'unit_amount' => 3900, // Raised from $29 to $39
    'currency' => 'usd',
    'recurring' => ['interval' => 'month'],
    'product' => 'prod_pro_plan',
    'lookup_key' => 'pro_monthly_v2',
]);

// Keep old price active but not visible
// Don't archive old price - existing subs still use it

// New signups get new price
public function subscribe(Request $request)
{
    $priceId = 'price_pro_monthly_v2'; // New price

    return $request->user()
        ->newSubscription('default', $priceId)
        ->checkout();
}

// Existing customers stay on their price
// Only changes if they cancel and re-subscribe

// Optional: Migrate to new pricing
public function migrateToNewPricing(User $user)
{
    $user->subscription('default')
        ->swap('price_pro_monthly_v2');
}
```
</grandfathering>

<regional_pricing>
## Regional / PPP Pricing

**Best for:** Global products where purchasing power varies.

**Example:** Full price in US, 50% in Brazil, 70% in EU.

**Implementation:**
```php
// config/stripe.php
'regional_pricing' => [
    'US' => ['multiplier' => 1.0, 'currency' => 'usd'],
    'BR' => ['multiplier' => 0.5, 'currency' => 'usd'], // 50% of US price
    'IN' => ['multiplier' => 0.4, 'currency' => 'usd'],
    'EU' => ['multiplier' => 0.9, 'currency' => 'eur'],
],

// Detect country and show price
public function showPricing(Request $request)
{
    $country = $this->detectCountry($request->ip());
    $regional = config("stripe.regional_pricing.{$country}", ['multiplier' => 1.0]);

    $basePrice = 29;
    $price = $basePrice * $regional['multiplier'];

    return view('pricing', [
        'price' => $price,
        'currency' => $regional['currency'],
        'country' => $country,
    ]);
}

// Create regional checkout
public function checkout(Request $request)
{
    $country = $request->country;
    $regional = config("stripe.regional_pricing.{$country}");

    $session = Stripe::checkout->sessions->create([
        'line_items' => [[
            'price_data' => [
                'currency' => $regional['currency'],
                'product' => 'prod_pro_plan',
                'unit_amount' => (int) (2900 * $regional['multiplier']),
                'recurring' => ['interval' => 'month'],
            ],
            'quantity' => 1,
        ]],
        'mode' => 'subscription',
        'metadata' => ['region' => $country],
    ]);
}
```

**Note:** Create separate prices in Stripe for each region to track revenue properly.
</regional_pricing>

<pricing_psychology>
## Pricing Psychology Tips

**1. Anchor with highest tier:**
Show Enterprise first to make Pro look reasonable.

**2. Highlight "most popular":**
```html
<div class="plan pro highlighted">
    <span class="badge">Most Popular</span>
    ...
</div>
```

**3. Use odd pricing:**
$29/month feels cheaper than $30/month.

**4. Show value, not just features:**
```html
<!-- Instead of: "10 projects" -->
<!-- Use: "Enough for your whole team" -->
```

**5. Reduce perceived risk:**
```html
<p>30-day money-back guarantee</p>
<p>Cancel anytime</p>
<p>No long-term contracts</p>
```

**6. Show annual savings prominently:**
```html
<span class="yearly-price">$24/mo</span>
<span class="savings">Save $58/year</span>
```

**7. Use social proof:**
```html
<p>Join 10,000+ teams using ProductName</p>
```
</pricing_psychology>
