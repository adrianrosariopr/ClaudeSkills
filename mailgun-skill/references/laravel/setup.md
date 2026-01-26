<overview>
Setup guide for Mailgun with Laravel using the built-in Mailgun driver via Symfony transport.
</overview>

<installation>
## Installation

```bash
composer require symfony/mailgun-mailer symfony/http-client
```

Both packages are required - `http-client` handles the API communication.
</installation>

<environment>
## Environment Variables

```env
MAIL_MAILER=mailgun
MAIL_FROM_ADDRESS=noreply@mail.yourdomain.com
MAIL_FROM_NAME="${APP_NAME}"

MAILGUN_DOMAIN=mail.yourdomain.com
MAILGUN_SECRET=key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MAILGUN_ENDPOINT=api.mailgun.net
MAILGUN_WEBHOOK_SIGNING_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

For EU region:
```env
MAILGUN_ENDPOINT=api.eu.mailgun.net
```
</environment>

<configuration>
## Configuration

<services_config>
**config/services.php:**
```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
    'webhook_signing_key' => env('MAILGUN_WEBHOOK_SIGNING_KEY'),
],
```
</services_config>

<mail_config>
**config/mail.php:**
```php
'default' => env('MAIL_MAILER', 'mailgun'),

'mailers' => [
    'mailgun' => [
        'transport' => 'mailgun',
        // Optional: increase timeout for large attachments
        // 'client' => [
        //     'timeout' => 60,
        // ],
    ],
    // ... other mailers
],

'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'noreply@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Your App'),
],
```
</mail_config>
</configuration>

<failover>
## Failover Configuration

Use multiple mailers with automatic failover:

```php
// config/mail.php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'mailgun',
            'ses',
            'smtp',
        ],
        'retry_after' => 60,
    ],
],
```

```env
MAIL_MAILER=failover
```
</failover>

<verification>
## Verify Setup

**Via Artisan Tinker:**
```bash
php artisan tinker
```

```php
Mail::raw('Test email from Laravel', function ($message) {
    $message->to('your-email@example.com')
            ->subject('Mailgun Test');
});
```

**Via Test Route (local only):**
```php
// routes/web.php
if (app()->environment('local')) {
    Route::get('/test-mailgun', function () {
        Mail::raw('Mailgun is working!', fn($m) =>
            $m->to('your-email@example.com')->subject('Test')
        );
        return 'Email sent! Check your inbox.';
    });
}
```

**Check Mailgun Logs:**
1. Go to Mailgun Dashboard → Sending → Logs
2. Find your test email
3. Confirm status: Accepted → Delivered
</verification>

<webhook_route>
## Webhook Route Setup

Exclude webhook route from CSRF protection:

```php
// routes/web.php
Route::post('/webhooks/mailgun', [MailgunWebhookController::class, 'handle'])
    ->withoutMiddleware([\App\Http\Middleware\VerifyCsrfToken::class]);
```

Or add to CSRF exceptions:
```php
// app/Http/Middleware/VerifyCsrfToken.php
protected $except = [
    'webhooks/mailgun',
];
```
</webhook_route>
