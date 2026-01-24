<overview>
Laravel Stripe setup using Laravel Cashier. Provides the most integrated experience with Eloquent models, migrations, and built-in webhook handling.
</overview>

<installation>
```bash
composer require laravel/cashier
php artisan vendor:publish --tag="cashier-migrations"
php artisan migrate
```
</installation>

<user_model>
```php
<?php

namespace App\Models;

use Laravel\Cashier\Billable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Billable;
}
```
</user_model>

<environment>
```env
STRIPE_KEY=pk_test_xxx
STRIPE_SECRET=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# Price IDs
STRIPE_PRICE_BASIC_MONTHLY=price_xxx
STRIPE_PRICE_BASIC_YEARLY=price_xxx
STRIPE_PRICE_PRO_MONTHLY=price_xxx
STRIPE_PRICE_PRO_YEARLY=price_xxx

# Optional
CASHIER_CURRENCY=usd
STRIPE_TRIAL_DAYS=7
```
</environment>

<config_file>
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
</config_file>

<webhook_route>
Cashier auto-registers `/stripe/webhook`. Exclude from CSRF:

**Laravel 11+ (bootstrap/app.php):**
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```
</webhook_route>

<verify_setup>
```bash
php artisan config:clear
php artisan tinker
>>> $user = User::first();
>>> $user->createAsStripeCustomer();
>>> $user->stripe_id  // Should show cus_xxx
```
</verify_setup>

<database_tables>
Cashier creates:
- `customers` columns on users table (stripe_id, pm_type, pm_last_four, trial_ends_at)
- `subscriptions` table
- `subscription_items` table
</database_tables>

<context7_queries>
For latest Laravel Cashier patterns:
```
mcp__context7__query-docs:
  libraryId: "/laravel/cashier-stripe"
  query: "your specific question"
```
</context7_queries>
