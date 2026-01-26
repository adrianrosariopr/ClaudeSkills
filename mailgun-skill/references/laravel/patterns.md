<overview>
Common patterns for using Mailgun with Laravel - Mailables, queuing, webhooks, notifications.
</overview>

<sending_basic>
## Sending Emails

<raw_email>
**Quick raw email:**
```php
use Illuminate\Support\Facades\Mail;

Mail::raw('This is a test email', function ($message) {
    $message->to('user@example.com')
            ->subject('Test Email');
});
```
</raw_email>

<with_mailable>
**Using Mailables (recommended):**

Create a Mailable:
```bash
php artisan make:mail OrderConfirmation
```

```php
// app/Mail/OrderConfirmation.php
namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class OrderConfirmation extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(
        public Order $order
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            from: new Address('orders@yourdomain.com', 'Your Store'),
            subject: "Order #{$this->order->id} Confirmed",
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'emails.orders.confirmation',
            with: [
                'orderItems' => $this->order->items,
                'total' => $this->order->total,
            ],
        );
    }
}
```

Send it:
```php
use App\Mail\OrderConfirmation;
use Illuminate\Support\Facades\Mail;

Mail::to($user->email)->send(new OrderConfirmation($order));
```
</with_mailable>

<multiple_recipients>
**Multiple recipients:**
```php
Mail::to(['user1@example.com', 'user2@example.com'])
    ->cc('manager@example.com')
    ->bcc('archive@example.com')
    ->send(new TeamNotification());

// Or with user objects
Mail::to($user)
    ->cc($managers)
    ->bcc($admins)
    ->send(new Notification());
```
</multiple_recipients>
</sending_basic>

<attachments>
## Attachments

```php
// app/Mail/InvoiceEmail.php
use Illuminate\Mail\Mailables\Attachment;

public function attachments(): array
{
    return [
        // From file path
        Attachment::fromPath('/path/to/invoice.pdf')
            ->as('invoice.pdf')
            ->withMime('application/pdf'),

        // From storage
        Attachment::fromStorage('invoices/invoice.pdf'),

        // From S3/cloud storage
        Attachment::fromStorageDisk('s3', 'invoices/invoice.pdf'),

        // From raw data
        Attachment::fromData(fn () => $pdfContent, 'invoice.pdf')
            ->withMime('application/pdf'),
    ];
}
```
</attachments>

<queuing>
## Queued Emails

**Queue for background sending:**
```php
Mail::to($user)->queue(new OrderConfirmation($order));
```

**Delayed sending:**
```php
Mail::to($user)
    ->later(now()->addMinutes(10), new OrderConfirmation($order));
```

**Specific queue:**
```php
Mail::to($user)
    ->onQueue('emails')
    ->queue(new OrderConfirmation($order));
```

**Make Mailable queueable by default:**
```php
class OrderConfirmation extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    public $tries = 3;
    public $backoff = [60, 300, 900]; // 1min, 5min, 15min

    public function failed(\Throwable $exception): void
    {
        Log::error('Order confirmation email failed', [
            'order_id' => $this->order->id,
            'error' => $exception->getMessage()
        ]);
    }
}
```
</queuing>

<custom_headers>
## Custom Headers and Mailgun Features

```php
// In your Mailable
public function build()
{
    return $this->view('emails.notification')
        ->withSymfonyMessage(function ($message) {
            $message->getHeaders()
                ->addTextHeader('X-Mailgun-Tag', 'notifications')
                ->addTextHeader('X-Mailgun-Variables', json_encode([
                    'user_id' => $this->user->id,
                    'campaign' => 'welcome'
                ]));
        });
}
```

Or using the headers method:
```php
use Illuminate\Mail\Mailables\Headers;

public function headers(): Headers
{
    return new Headers(
        text: [
            'X-Mailgun-Tag' => 'notifications',
            'X-Custom-Header' => 'custom-value',
        ],
    );
}
```
</custom_headers>

<webhook_controller>
## Webhook Controller

```php
// app/Http/Controllers/MailgunWebhookController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use App\Models\User;

class MailgunWebhookController extends Controller
{
    public function handle(Request $request)
    {
        if (!$this->verifySignature($request)) {
            return response()->json(['error' => 'Invalid signature'], 401);
        }

        $eventData = $request->input('event-data');
        $event = $eventData['event'];
        $recipient = $eventData['recipient'] ?? null;

        Log::info("Mailgun webhook: {$event}", ['recipient' => $recipient]);

        match ($event) {
            'failed' => $this->handleFailure($eventData),
            'complained' => $this->handleComplaint($recipient),
            'unsubscribed' => $this->handleUnsubscribe($recipient),
            default => null,
        };

        return response()->json(['received' => true]);
    }

    private function verifySignature(Request $request): bool
    {
        $signature = $request->input('signature');
        $timestamp = $signature['timestamp'] ?? '';
        $token = $signature['token'] ?? '';
        $providedSignature = $signature['signature'] ?? '';

        $expectedSignature = hash_hmac(
            'sha256',
            $timestamp . $token,
            config('services.mailgun.webhook_signing_key')
        );

        return hash_equals($expectedSignature, $providedSignature);
    }

    private function handleFailure(array $eventData): void
    {
        if (($eventData['severity'] ?? '') !== 'permanent') return;

        User::where('email', $eventData['recipient'])->update([
            'email_status' => 'bounced',
            'can_email' => false,
        ]);
    }

    private function handleComplaint(string $email): void
    {
        User::where('email', $email)->update([
            'email_status' => 'complained',
            'can_email' => false,
        ]);
    }

    private function handleUnsubscribe(string $email): void
    {
        User::where('email', $email)->update(['subscribed' => false]);
    }
}
```
</webhook_controller>

<direct_api>
## Direct API Access

For features not supported by Laravel's Mail facade:

```php
use Illuminate\Support\Facades\Http;

$domain = config('services.mailgun.domain');
$secret = config('services.mailgun.secret');
$endpoint = config('services.mailgun.endpoint');

// Send with Mailgun template
$response = Http::withBasicAuth('api', $secret)
    ->asForm()
    ->post("https://{$endpoint}/v3/{$domain}/messages", [
        'from' => "noreply@{$domain}",
        'to' => 'user@example.com',
        'subject' => 'Welcome',
        'template' => 'welcome-email',
        'h:X-Mailgun-Variables' => json_encode([
            'name' => 'John',
            'plan' => 'premium'
        ])
    ]);

if ($response->successful()) {
    $messageId = $response->json('id');
}
```
</direct_api>

<testing>
## Testing

**Prevent actual sending in tests:**
```php
Mail::fake();

$user->sendWelcomeEmail();

Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
    return $mail->hasTo($user->email);
});

Mail::assertSent(WelcomeEmail::class, 1);
Mail::assertNotSent(InvoiceEmail::class);
```

**Preview emails in browser:**
```php
// routes/web.php (only in local environment)
if (app()->environment('local')) {
    Route::get('/mail-preview', function () {
        $order = Order::factory()->make();
        return new App\Mail\OrderConfirmation($order);
    });
}
```
</testing>

<error_handling>
## Error Handling

```php
use Symfony\Component\Mailer\Exception\TransportExceptionInterface;

try {
    Mail::to($user)->send(new OrderConfirmation($order));
} catch (TransportExceptionInterface $e) {
    Log::error('Email failed', [
        'recipient' => $user->email,
        'error' => $e->getMessage()
    ]);
}
```
</error_handling>

<notifications>
## Using with Laravel Notifications

```php
// app/Notifications/OrderShipped.php
use Illuminate\Notifications\Notification;
use App\Mail\OrderShippedMail;

class OrderShipped extends Notification
{
    public function __construct(
        public Order $order
    ) {}

    public function via($notifiable): array
    {
        return ['mail'];
    }

    public function toMail($notifiable): OrderShippedMail
    {
        return (new OrderShippedMail($this->order))
            ->to($notifiable->email);
    }
}

// Usage
$user->notify(new OrderShipped($order));
```
</notifications>
