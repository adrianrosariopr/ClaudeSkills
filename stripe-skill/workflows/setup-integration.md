# Workflow: Set Up Stripe Integration

<required_reading>
**Read these reference files NOW:**
1. references/laravel-cashier.md
2. references/security.md
</required_reading>

<process>

## Step 1: Install Laravel Cashier

```bash
composer require laravel/cashier
```

Verify installation:
```bash
composer show laravel/cashier
```

## Step 2: Run Cashier Migrations

Publish and run the migrations:

```bash
php artisan vendor:publish --tag="cashier-migrations"
php artisan migrate
```

This creates:
- `customers` table (Stripe customer data)
- `subscriptions` table
- `subscription_items` table

## Step 3: Add Billable Trait to User Model

Edit `app/Models/User.php`:

```php
<?php

namespace App\Models;

use Laravel\Cashier\Billable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Billable;

    // ... rest of model
}
```

## Step 4: Get Stripe API Keys

1. Go to [Stripe Dashboard](https://dashboard.stripe.com)
2. Click **Developers** → **API keys**
3. Copy your **Publishable key** (`pk_test_...`)
4. Copy your **Secret key** (`sk_test_...`)

**For local development, use test mode keys.**

## Step 5: Configure Environment Variables

Add to `.env`:

```env
# Stripe API Keys
STRIPE_KEY=pk_test_your_publishable_key
STRIPE_SECRET=sk_test_your_secret_key

# Will be set after creating webhook in Step 7
STRIPE_WEBHOOK_SECRET=

# Optional: Cashier settings
CASHIER_CURRENCY=usd
CASHIER_CURRENCY_LOCALE=en
```

## Step 6: Publish Cashier Config (Optional)

```bash
php artisan vendor:publish --tag="cashier-config"
```

This creates `config/cashier.php` for customization.

## Step 7: Create Stripe Products and Prices

In Stripe Dashboard → **Products**:

1. Click **+ Add product**
2. Enter product name and description
3. Add pricing (monthly/yearly)
4. Save and copy the **Price ID** (`price_...`)

Add Price IDs to `.env`:

```env
STRIPE_PRICE_BASIC_MONTHLY=price_xxxxx
STRIPE_PRICE_BASIC_YEARLY=price_xxxxx
STRIPE_PRICE_PRO_MONTHLY=price_xxxxx
STRIPE_PRICE_PRO_YEARLY=price_xxxxx
```

## Step 8: Create Stripe Config File

Create `config/stripe.php`:

```php
<?php

return [
    'key' => env('STRIPE_KEY'),
    'secret' => env('STRIPE_SECRET'),
    'webhook_secret' => env('STRIPE_WEBHOOK_SECRET'),

    'prices' => [
        'basic_monthly' => env('STRIPE_PRICE_BASIC_MONTHLY'),
        'basic_yearly' => env('STRIPE_PRICE_BASIC_YEARLY'),
        'pro_monthly' => env('STRIPE_PRICE_PRO_MONTHLY'),
        'pro_yearly' => env('STRIPE_PRICE_PRO_YEARLY'),
    ],

    'trial_days' => env('STRIPE_TRIAL_DAYS', 7),
];
```

## Step 9: Create Webhook Endpoint

Use Cashier's artisan command:

```bash
php artisan cashier:webhook --disabled
```

Or manually in Stripe Dashboard → **Developers** → **Webhooks**:
1. Click **Add endpoint**
2. URL: `https://yourdomain.com/stripe/webhook`
3. Select events (see webhooks workflow)
4. Copy **Signing secret** to `.env` as `STRIPE_WEBHOOK_SECRET`

**For local development:**
```bash
stripe listen --forward-to localhost:8000/stripe/webhook
```

## Step 10: Verify Setup

```bash
# Clear and cache config
php artisan config:clear
php artisan config:cache

# Test Stripe connection
php artisan tinker
>>> $user = User::first();
>>> $user->createAsStripeCustomer();
>>> $user->stripe_id  // Should show cus_xxxxx
```

</process>

<success_criteria>
Setup is complete when:
- [ ] Laravel Cashier installed via Composer
- [ ] Migrations run (customers, subscriptions tables exist)
- [ ] User model has Billable trait
- [ ] `.env` has STRIPE_KEY and STRIPE_SECRET
- [ ] Products/Prices created in Stripe Dashboard
- [ ] Price IDs added to config
- [ ] Webhook endpoint configured (or Stripe CLI for local)
- [ ] Test user can be created as Stripe customer
</success_criteria>

<anti_patterns>
Avoid:
- Committing API keys to version control
- Using live keys during development
- Skipping webhook setup (webhooks are essential)
- Hardcoding Price IDs (use config/env)
</anti_patterns>
