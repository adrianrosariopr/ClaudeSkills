<overview>
Common patterns for using Mailgun with Node.js - sending emails, handling webhooks, API usage.
</overview>

<sending_messages>
## Sending Messages

<basic_email>
**Basic email:**
```javascript
const { mg, domain } = require('./lib/mailgun');

const result = await mg.messages.create(domain, {
  from: 'Sender Name <sender@yourdomain.com>',
  to: ['recipient@example.com'],
  subject: 'Hello',
  text: 'Plain text version',
  html: '<h1>HTML version</h1>'
});

console.log(result.id);     // Message ID
console.log(result.message); // "Queued. Thank you."
```
</basic_email>

<with_attachments>
**With attachments:**
```javascript
const fs = require('fs').promises;

const fileData = await fs.readFile('./invoice.pdf');

await mg.messages.create(domain, {
  from: 'sender@yourdomain.com',
  to: ['recipient@example.com'],
  subject: 'Your Invoice',
  text: 'Please find your invoice attached.',
  attachment: [{
    filename: 'invoice.pdf',
    data: fileData
  }]
});
```
</with_attachments>

<with_template>
**Using Mailgun templates:**
```javascript
await mg.messages.create(domain, {
  from: 'sender@yourdomain.com',
  to: ['recipient@example.com'],
  subject: 'Welcome!',
  template: 'welcome-email',
  'h:X-Mailgun-Variables': JSON.stringify({
    firstName: 'John',
    accountType: 'premium'
  })
});
```
</with_template>

<batch_sending>
**Batch sending with personalization:**
```javascript
await mg.messages.create(domain, {
  from: 'sender@yourdomain.com',
  to: ['alice@example.com', 'bob@example.com'],
  subject: 'Hello %recipient.name%',
  text: 'Hi %recipient.name%, your code is %recipient.code%',
  'recipient-variables': JSON.stringify({
    'alice@example.com': { name: 'Alice', code: 'ABC123' },
    'bob@example.com': { name: 'Bob', code: 'XYZ789' }
  })
});
```
</batch_sending>

<all_options>
**All message options:**
```javascript
await mg.messages.create(domain, {
  // Required
  from: 'sender@domain.com',
  to: ['recipient@example.com'],

  // Content (at least one required)
  subject: 'Subject line',
  text: 'Plain text body',
  html: '<p>HTML body</p>',
  template: 'template-name',

  // Recipients
  cc: ['cc@example.com'],
  bcc: ['bcc@example.com'],

  // Attachments
  attachment: [{ filename: 'file.pdf', data: buffer }],
  inline: [{ filename: 'logo.png', data: buffer }],

  // Template variables
  'h:X-Mailgun-Variables': JSON.stringify({ key: 'value' }),
  'recipient-variables': JSON.stringify({ 'email': { var: 'val' } }),

  // Delivery options
  'o:deliverytime': new Date(Date.now() + 3600000).toUTCString(),
  'o:testmode': 'yes',
  'o:tag': ['tag1', 'tag2'],

  // Tracking
  'o:tracking': 'yes',
  'o:tracking-clicks': 'yes',
  'o:tracking-opens': 'yes',

  // Custom headers
  'h:Reply-To': 'reply@example.com',
  'h:X-Custom-Header': 'custom-value',

  // Custom variables (for webhooks)
  'v:user-id': '12345',
  'v:campaign': 'welcome-series'
});
```
</all_options>
</sending_messages>

<webhook_handler>
## Webhook Handler (Express)

```javascript
const express = require('express');
const crypto = require('crypto');
const router = express.Router();

function verifyWebhook(timestamp, token, signature) {
  const encodedToken = crypto
    .createHmac('sha256', process.env.MAILGUN_WEBHOOK_SIGNING_KEY)
    .update(timestamp.concat(token))
    .digest('hex');
  return encodedToken === signature;
}

router.post('/webhooks/mailgun', express.json(), async (req, res) => {
  const { signature, 'event-data': eventData } = req.body;

  // Verify signature
  if (!verifyWebhook(signature.timestamp, signature.token, signature.signature)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = eventData.event;
  const recipient = eventData.recipient;

  switch (event) {
    case 'failed':
      if (eventData.severity === 'permanent') {
        await handleBounce(recipient, eventData);
      }
      break;
    case 'complained':
      await handleComplaint(recipient);
      break;
    case 'unsubscribed':
      await handleUnsubscribe(recipient);
      break;
  }

  res.status(200).json({ received: true });
});
```
</webhook_handler>

<api_methods>
## API Methods

<domains_api>
**Domains:**
```javascript
const domains = await mg.domains.list();
const domain = await mg.domains.get('mail.yourdomain.com');
await mg.domains.verify('mail.yourdomain.com');
```
</domains_api>

<suppressions_api>
**Suppressions:**
```javascript
// List
const bounces = await mg.suppressions.list(domain, 'bounces');
const complaints = await mg.suppressions.list(domain, 'complaints');

// Check specific address
const bounce = await mg.suppressions.get(domain, 'bounces', 'user@example.com');

// Add
await mg.suppressions.create(domain, 'bounces', {
  address: 'user@example.com',
  code: 550,
  error: 'Mailbox not found'
});

// Remove
await mg.suppressions.destroy(domain, 'bounces', 'user@example.com');
```
</suppressions_api>

<validation_api>
**Email Validation (requires Optimize subscription):**
```javascript
const result = await mg.validate.get('user@example.com');

console.log(result.is_valid);        // true/false
console.log(result.risk);            // 'low', 'medium', 'high'
console.log(result.is_disposable);   // true/false
console.log(result.is_role_address); // true/false
```
</validation_api>

<events_api>
**Events:**
```javascript
const events = await mg.events.get(domain, {
  begin: new Date(Date.now() - 86400000).toUTCString(),
  ascending: 'yes',
  limit: 100,
  event: 'failed' // Filter by event type
});
```
</events_api>
</api_methods>

<error_handling>
## Error Handling

```javascript
try {
  const result = await mg.messages.create(domain, messageData);
} catch (error) {
  console.error('Status:', error.status);
  console.error('Message:', error.message);

  if (error.status === 401) {
    // Invalid API key or wrong region
  } else if (error.status === 400) {
    // Invalid request parameters
  } else if (error.status === 429) {
    // Rate limited - implement backoff
  }
}
```

<retry_pattern>
**Exponential backoff:**
```javascript
async function sendWithRetry(messageData, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await mg.messages.create(domain, messageData);
    } catch (error) {
      if (error.status === 429 && attempt < maxRetries - 1) {
        const delay = Math.pow(2, attempt) * 1000;
        await new Promise(r => setTimeout(r, delay));
      } else {
        throw error;
      }
    }
  }
}
```
</retry_pattern>
</error_handling>

<email_service_pattern>
## Email Service Pattern

```javascript
// services/emailService.js
const { mg, domain } = require('../lib/mailgun');

class EmailService {
  async sendWelcome(user) {
    return this.send({
      to: user.email,
      template: 'welcome',
      variables: { name: user.name }
    });
  }

  async sendPasswordReset(user, token) {
    return this.send({
      to: user.email,
      template: 'password-reset',
      variables: {
        name: user.name,
        resetUrl: `${process.env.APP_URL}/reset?token=${token}`
      }
    });
  }

  async send({ to, template, variables, subject }) {
    try {
      const result = await mg.messages.create(domain, {
        from: `${process.env.APP_NAME} <noreply@${domain}>`,
        to: Array.isArray(to) ? to : [to],
        subject,
        template,
        'h:X-Mailgun-Variables': JSON.stringify(variables)
      });
      return { success: true, id: result.id };
    } catch (error) {
      console.error('Email failed:', error.message);
      return { success: false, error: error.message };
    }
  }
}

module.exports = new EmailService();
```
</email_service_pattern>
