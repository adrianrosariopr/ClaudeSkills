# Workflow: Setup Mailgun

<required_reading>
**Read these reference files NOW:**
1. references/dns-configuration.md
2. references/api-keys-security.md
3. references/nodejs-sdk.md (if Node.js)
4. references/laravel-driver.md (if Laravel)
</required_reading>

<process>
## Step 1: Create Mailgun Account and Domain

1. Sign up at https://www.mailgun.com
2. Go to **Sending → Domains → Add New Domain**
3. Use a subdomain: `mail.yourdomain.com` (NOT root domain)
4. Select region: US or EU (affects API endpoint)

**Important:** Free tier limits:
- 100 emails/day (recently reduced from 5,000/month)
- Sandbox domains can only send to verified recipients

## Step 2: Configure DNS Records

Mailgun provides DNS records to add. Go to your DNS provider and add:

**SPF Record (TXT):**
```
Type: TXT
Host: mail (or your subdomain)
Value: v=spf1 include:mailgun.org ~all
```

If you have existing SPF, merge it:
```
v=spf1 ip4:your.ip include:existingservice.com include:mailgun.org ~all
```

**DKIM Record (TXT):**
```
Type: TXT
Host: selector._domainkey.mail (Mailgun provides exact value)
Value: (long key string from Mailgun dashboard)
```

**MX Records (for receiving):**
```
Type: MX
Host: mail
Value: mxa.mailgun.org (priority 10)
Value: mxb.mailgun.org (priority 10)
```

**Tracking CNAME (for open/click tracking):**
```
Type: CNAME
Host: email.mail (Mailgun provides exact value)
Value: mailgun.org
```

**Warning:** If using Cloudflare, disable proxy (orange cloud) for these records.

## Step 3: Verify Domain

1. Return to Mailgun dashboard
2. Click **Verify DNS Settings**
3. Wait for all green checkmarks (can take 24-48 hours)

## Step 4: Get API Credentials

1. Go to **Account Settings → API Security**
2. Copy your **Private API Key** (starts with `key-`)
3. Note your **domain** and **region**

## Step 5: Install SDK

### Node.js

```bash
npm install mailgun.js form-data
```

Create `.env`:
```env
MAILGUN_API_KEY=key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MAILGUN_DOMAIN=mail.yourdomain.com
MAILGUN_REGION=us  # or 'eu'
```

Create `lib/mailgun.js`:
```javascript
const formData = require('form-data');
const Mailgun = require('mailgun.js');

const mailgun = new Mailgun(formData);

const mg = mailgun.client({
  username: 'api',
  key: process.env.MAILGUN_API_KEY,
  url: process.env.MAILGUN_REGION === 'eu'
    ? 'https://api.eu.mailgun.net'
    : 'https://api.mailgun.net'
});

module.exports = { mg, domain: process.env.MAILGUN_DOMAIN };
```

### Laravel

```bash
composer require symfony/mailgun-mailer symfony/http-client
```

Update `.env`:
```env
MAIL_MAILER=mailgun
MAILGUN_DOMAIN=mail.yourdomain.com
MAILGUN_SECRET=key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MAILGUN_ENDPOINT=api.mailgun.net  # or api.eu.mailgun.net for EU
```

Update `config/services.php`:
```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```

Update `config/mail.php`:
```php
'default' => env('MAIL_MAILER', 'mailgun'),
```

## Step 6: Test Email

### Node.js

Create `test-email.js`:
```javascript
require('dotenv').config();
const { mg, domain } = require('./lib/mailgun');

async function testEmail() {
  try {
    const result = await mg.messages.create(domain, {
      from: `Test <mailgun@${domain}>`,
      to: ['your-email@example.com'],
      subject: 'Mailgun Test',
      text: 'If you receive this, Mailgun is working!',
      html: '<h1>Success!</h1><p>Mailgun is configured correctly.</p>'
    });
    console.log('Email sent:', result);
  } catch (error) {
    console.error('Error:', error.message);
  }
}

testEmail();
```

Run: `node test-email.js`

### Laravel

```bash
php artisan tinker
```

```php
Mail::raw('Mailgun test from Laravel!', function ($message) {
    $message->to('your-email@example.com')
            ->subject('Mailgun Test');
});
```

Or create a test route:
```php
Route::get('/test-email', function () {
    Mail::raw('Test email', fn($m) => $m->to('test@example.com')->subject('Test'));
    return 'Email sent!';
});
```

## Step 7: Verify in Dashboard

1. Go to **Sending → Logs**
2. Find your test email
3. Confirm status: **Accepted** → **Delivered**

If status shows **Failed**, check error message and refer to `workflows/debug-delivery.md`.
</process>

<success_criteria>
Setup is complete when:
- [ ] Domain added to Mailgun
- [ ] All DNS records configured and verified (green checkmarks)
- [ ] SDK installed and configured
- [ ] API key stored securely in environment variables
- [ ] Test email sent and delivered successfully
- [ ] Email appears in Mailgun logs with "Delivered" status
</success_criteria>

<anti_patterns>
Avoid:
- Using root domain instead of subdomain
- Hardcoding API keys in source code
- Skipping DNS verification (emails will go to spam)
- Using sandbox domain in production
- Forgetting to configure region (US vs EU mismatch causes 401 errors)
</anti_patterns>
