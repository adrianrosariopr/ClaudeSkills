<overview>
Stripe payment integration requires careful attention to security. Key concerns: API key protection, webhook verification, PCI compliance, and secure data handling.
</overview>

<pci_compliance>
**What is PCI DSS?**
Payment Card Industry Data Security Standard - rules for handling card data.

**How Checkout maintains compliance:**
- Card data entered on Stripe's servers
- Your server never sees raw card numbers
- Stripe is PCI Level 1 certified

**Your responsibilities:**
- Use HTTPS everywhere
- Don't log card data (even masked)
- Use Stripe Checkout or Elements
- Don't build custom card forms
</pci_compliance>

<api_key_security>
**Never commit API keys:**
```bash
# .gitignore
.env
.env.*
```

**Store in environment:**
```env
STRIPE_KEY=pk_test_xxxxx
STRIPE_SECRET=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
```

**Access via config:**
```php
config('cashier.key');
config('cashier.secret');
```

**Restricted keys (for production):**
- Stripe Dashboard → Developers → API keys → Restricted keys
- Create key with only needed permissions
- Use for specific services (e.g., webhook-only key)
</api_key_security>

<webhook_security>
**Always verify signatures:**
Laravel Cashier does this automatically. If handling manually:

```php
$payload = @file_get_contents('php://input');
$sigHeader = $_SERVER['HTTP_STRIPE_SIGNATURE'];
$secret = config('cashier.webhook.secret');

try {
    $event = \Stripe\Webhook::constructEvent(
        $payload,
        $sigHeader,
        $secret
    );
} catch (\Stripe\Exception\SignatureVerificationException $e) {
    return response('Invalid signature', 400);
}
```

**Why this matters:**
- Prevents forged webhooks
- Confirms event came from Stripe
- Required for secure payment processing

**Common mistakes:**
- Using parsed JSON instead of raw body (breaks signature)
- Wrong webhook secret (test vs live)
- Missing CSRF exclusion (can't receive webhooks)
</webhook_security>

<https_requirement>
**Stripe requires HTTPS for:**
- Checkout redirects
- Webhook endpoints
- Customer Portal

**Local development:**
- Use `http://localhost` (allowed in test mode)
- Or use ngrok for HTTPS tunnel

**Production:**
- SSL certificate required
- Redirect all HTTP to HTTPS
- HSTS header recommended
</https_requirement>

<input_validation>
**Validate price keys:**
```php
$request->validate([
    'price' => 'required|string|in:basic_monthly,basic_yearly,pro_monthly,pro_yearly',
]);
```

**Never trust client-side data:**
```php
// BAD: Using client-provided price ID directly
$priceId = $request->price_id;

// GOOD: Map to known server-side values
$prices = [
    'basic_monthly' => config('stripe.prices.basic_monthly'),
    'pro_monthly' => config('stripe.prices.pro_monthly'),
];
$priceId = $prices[$request->price] ?? throw new InvalidArgumentException();
```

**Validate quantities:**
```php
$request->validate([
    'quantity' => 'integer|min:1|max:100',
]);
```
</input_validation>

<metadata_security>
**Don't store sensitive data in metadata:**
```php
// BAD
'metadata' => [
    'password' => $user->password,
    'ssn' => $user->ssn,
]

// GOOD
'metadata' => [
    'user_id' => $user->id,
    'product' => $productKey,
]
```

**Metadata is visible in:**
- Stripe Dashboard
- API responses
- Webhook payloads
</metadata_security>

<logging>
**Log safely:**
```php
// BAD: Logging full webhook payload
Log::info('Webhook received', $payload);

// GOOD: Log only what you need
Log::info('Subscription created', [
    'subscription_id' => $subscription['id'],
    'customer' => $subscription['customer'],
    'status' => $subscription['status'],
]);
```

**Never log:**
- API keys
- Webhook secrets
- Full card numbers (even if somehow received)
- Customer PII unless necessary
</logging>

<idempotency>
**Prevent duplicate processing:**
```php
protected function handleInvoicePaymentSucceeded(array $payload): void
{
    $invoice = $payload['data']['object'];

    // Idempotency check
    if (Payment::where('stripe_invoice_id', $invoice['id'])->exists()) {
        Log::info('Already processed', ['id' => $invoice['id']]);
        return;
    }

    // Process once
    Payment::create([
        'stripe_invoice_id' => $invoice['id'],
        // ...
    ]);
}
```

**Why this matters:**
- Stripe may retry webhooks
- Network issues cause duplicates
- Prevents double-charging or double-fulfillment
</idempotency>

<csrf_exclusion>
**Webhooks can't include CSRF tokens.**

**Laravel 11+ (bootstrap/app.php):**
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

**Laravel 10 and earlier:**
```php
// app/Http/Middleware/VerifyCsrfToken.php
protected $except = [
    'stripe/*',
];
```

**Only exclude webhook routes, not billing pages.**
</csrf_exclusion>

<rate_limiting>
**Protect checkout endpoints:**
```php
// routes/web.php
Route::middleware(['auth', 'throttle:checkout'])->group(function () {
    Route::post('/subscription/checkout', [SubscriptionController::class, 'checkout']);
});

// RouteServiceProvider or bootstrap/app.php
RateLimiter::for('checkout', function (Request $request) {
    return Limit::perMinute(5)->by($request->user()?->id ?: $request->ip());
});
```
</rate_limiting>

<error_handling>
**Don't expose internal errors:**
```php
// BAD
catch (\Exception $e) {
    return response()->json(['error' => $e->getMessage()], 500);
}

// GOOD
catch (\Stripe\Exception\CardException $e) {
    return back()->with('error', 'Your card was declined. Please try another card.');
} catch (\Stripe\Exception\ApiErrorException $e) {
    Log::error('Stripe API error', ['message' => $e->getMessage()]);
    return back()->with('error', 'Payment processing is temporarily unavailable.');
}
```
</error_handling>

<security_checklist>
**Before going live:**
- [ ] API keys in environment variables (not code)
- [ ] `.env` in `.gitignore`
- [ ] HTTPS enabled
- [ ] Webhook signature verification enabled
- [ ] CSRF protection excludes only webhook routes
- [ ] Price keys validated server-side
- [ ] No sensitive data in logs or metadata
- [ ] Idempotent webhook handlers
- [ ] Error messages don't expose internals
- [ ] Rate limiting on checkout endpoints
- [ ] Using Stripe Checkout (not custom card forms)
</security_checklist>
