# Workflow: Send Emails

<required_reading>
**Read these reference files NOW:**
1. references/sending-emails.md
2. references/templates-personalization.md
3. references/nodejs-sdk.md (if Node.js)
4. references/laravel-driver.md (if Laravel)
</required_reading>

<process>
## Step 1: Determine Email Type

**Transactional:** One-to-one emails triggered by user actions
- Password resets, order confirmations, notifications
- Use `messages.create()` or Laravel `Mail::send()`

**Bulk/Batch:** Same email to multiple recipients with personalization
- Newsletters, announcements
- Use recipient variables for personalization

**Template-based:** Pre-designed templates stored in Mailgun
- Create in Mailgun dashboard, send via API
- Good for consistent branding

## Step 2: Send Basic Email

### Node.js

```javascript
const { mg, domain } = require('./lib/mailgun');

async function sendEmail({ to, subject, text, html }) {
  try {
    const result = await mg.messages.create(domain, {
      from: `Your App <noreply@${domain}>`,
      to: Array.isArray(to) ? to : [to],
      subject,
      text,
      html
    });
    return { success: true, id: result.id };
  } catch (error) {
    console.error('Mailgun error:', error.message);
    return { success: false, error: error.message };
  }
}

// Usage
await sendEmail({
  to: 'user@example.com',
  subject: 'Welcome!',
  text: 'Thanks for signing up.',
  html: '<h1>Welcome!</h1><p>Thanks for signing up.</p>'
});
```

### Laravel

Create a Mailable:
```bash
php artisan make:mail WelcomeEmail
```

```php
// app/Mail/WelcomeEmail.php
namespace App\Mail;

use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;

class WelcomeEmail extends Mailable
{
    public function __construct(
        public string $userName
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Welcome to Our App',
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'emails.welcome',
            with: ['name' => $this->userName],
        );
    }
}
```

```php
// resources/views/emails/welcome.blade.php
<h1>Welcome, {{ $name }}!</h1>
<p>Thanks for signing up.</p>
```

Send:
```php
use App\Mail\WelcomeEmail;
use Illuminate\Support\Facades\Mail;

Mail::to('user@example.com')->send(new WelcomeEmail('John'));
```

## Step 3: Send with Attachments

### Node.js

```javascript
const fs = require('fs').promises;
const path = require('path');

async function sendWithAttachment({ to, subject, text, filePath }) {
  const fileData = await fs.readFile(filePath);
  const fileName = path.basename(filePath);

  return mg.messages.create(domain, {
    from: `Your App <noreply@${domain}>`,
    to: [to],
    subject,
    text,
    attachment: [{
      filename: fileName,
      data: fileData
    }]
  });
}

// Multiple attachments
attachment: [
  { filename: 'report.pdf', data: pdfBuffer },
  { filename: 'image.png', data: imageBuffer }
]
```

### Laravel

```php
class InvoiceEmail extends Mailable
{
    public function __construct(
        public string $invoicePath
    ) {}

    public function attachments(): array
    {
        return [
            Attachment::fromPath($this->invoicePath),
            // Or from storage
            Attachment::fromStorage('/invoices/invoice.pdf'),
        ];
    }
}
```

## Step 4: Use Mailgun Templates

### Create Template in Dashboard

1. Go to **Sending → Templates**
2. Click **Create Template**
3. Use Handlebars syntax for variables: `{{name}}`, `{{order_id}}`

### Send with Template (Node.js)

```javascript
await mg.messages.create(domain, {
  from: `Your App <noreply@${domain}>`,
  to: ['user@example.com'],
  subject: 'Your Order Confirmation',
  template: 'order-confirmation',  // template name
  'h:X-Mailgun-Variables': JSON.stringify({
    name: 'John',
    order_id: '12345',
    items: [
      { name: 'Widget', price: 9.99 },
      { name: 'Gadget', price: 19.99 }
    ]
  })
});
```

### Send with Template (Laravel)

Laravel uses its own Blade templates, but you can call Mailgun API directly:

```php
use Illuminate\Support\Facades\Http;

$response = Http::withBasicAuth('api', config('services.mailgun.secret'))
    ->asForm()
    ->post("https://api.mailgun.net/v3/{$domain}/messages", [
        'from' => "noreply@{$domain}",
        'to' => 'user@example.com',
        'subject' => 'Order Confirmation',
        'template' => 'order-confirmation',
        'h:X-Mailgun-Variables' => json_encode([
            'name' => 'John',
            'order_id' => '12345'
        ])
    ]);
```

## Step 5: Send Bulk Emails with Personalization

### Node.js - Recipient Variables

```javascript
// Send to multiple recipients with individual personalization
await mg.messages.create(domain, {
  from: `Your App <noreply@${domain}>`,
  to: ['alice@example.com', 'bob@example.com', 'carol@example.com'],
  subject: 'Hello %recipient.name%',
  text: 'Hi %recipient.name%, your code is %recipient.code%',
  'recipient-variables': JSON.stringify({
    'alice@example.com': { name: 'Alice', code: 'ABC123' },
    'bob@example.com': { name: 'Bob', code: 'DEF456' },
    'carol@example.com': { name: 'Carol', code: 'GHI789' }
  })
});
```

### Laravel - Loop with Queue

```php
use App\Mail\NewsletterEmail;
use Illuminate\Support\Facades\Mail;

$subscribers = User::where('subscribed', true)->get();

foreach ($subscribers as $user) {
    Mail::to($user->email)
        ->queue(new NewsletterEmail($user));
}
```

## Step 6: Schedule Emails for Later

### Node.js

```javascript
await mg.messages.create(domain, {
  from: `Your App <noreply@${domain}>`,
  to: ['user@example.com'],
  subject: 'Scheduled Email',
  text: 'This was scheduled!',
  'o:deliverytime': new Date(Date.now() + 3600000).toUTCString() // 1 hour from now
});
```

Maximum: 3 days in the future.

### Laravel

Use Laravel's queue with delay:

```php
Mail::to($user)
    ->later(now()->addHours(1), new ReminderEmail($user));
```

## Step 7: Enable Tracking

### Node.js

```javascript
await mg.messages.create(domain, {
  from: `Your App <noreply@${domain}>`,
  to: ['user@example.com'],
  subject: 'Tracked Email',
  html: '<p>Click <a href="https://example.com">here</a></p>',
  'o:tracking': 'yes',
  'o:tracking-clicks': 'yes',
  'o:tracking-opens': 'yes'
});
```

### Laravel

Configure in `config/mail.php` or per-message headers.
</process>

<success_criteria>
Email sending works when:
- [ ] Basic email delivered successfully
- [ ] Attachments arrive intact
- [ ] Template variables are substituted correctly
- [ ] Bulk emails personalized per recipient
- [ ] Scheduled emails delivered at correct time
- [ ] Tracking events appear in Mailgun logs
</success_criteria>

<anti_patterns>
Avoid:
- Using `text` without `html` (or vice versa) - provide both
- Sending bulk emails without recipient variables (all recipients visible to each other)
- Large attachments (>25MB) - use links instead
- Inline SVG images (not supported by many email clients)
- CSS classes in HTML (use inline styles only)
- Scheduling emails more than 3 days ahead
</anti_patterns>
