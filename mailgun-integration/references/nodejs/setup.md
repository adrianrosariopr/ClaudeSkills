<overview>
Setup guide for Mailgun with Node.js using the official mailgun.js SDK.
</overview>

<installation>
## Installation

```bash
npm install mailgun.js form-data
```

Both packages are required - `form-data` handles multipart form encoding.
</installation>

<environment>
## Environment Variables

```env
MAILGUN_API_KEY=key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MAILGUN_DOMAIN=mail.yourdomain.com
MAILGUN_REGION=us  # or 'eu'
MAILGUN_WEBHOOK_SIGNING_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
</environment>

<initialization>
## Client Initialization

```javascript
const formData = require('form-data');
const Mailgun = require('mailgun.js');

const mailgun = new Mailgun(formData);

// US region (default)
const mg = mailgun.client({
  username: 'api',
  key: process.env.MAILGUN_API_KEY
});

// EU region
const mgEU = mailgun.client({
  username: 'api',
  key: process.env.MAILGUN_API_KEY,
  url: 'https://api.eu.mailgun.net'
});
```
</initialization>

<reusable_module>
## Recommended: Reusable Module

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

const domain = process.env.MAILGUN_DOMAIN;

module.exports = { mg, domain };
```

Usage:
```javascript
const { mg, domain } = require('./lib/mailgun');
```
</reusable_module>

<typescript>
## TypeScript Setup

```typescript
import formData from 'form-data';
import Mailgun from 'mailgun.js';
import { IMailgunClient } from 'mailgun.js/Interfaces';

const mailgun = new Mailgun(formData);

const mg: IMailgunClient = mailgun.client({
  username: 'api',
  key: process.env.MAILGUN_API_KEY!
});

interface MessageData {
  from: string;
  to: string[];
  subject: string;
  text?: string;
  html?: string;
}

async function sendEmail(data: MessageData) {
  return mg.messages.create(process.env.MAILGUN_DOMAIN!, data);
}
```
</typescript>

<verification>
## Verify Setup

Create `test-mailgun.js`:

```javascript
require('dotenv').config();
const { mg, domain } = require('./lib/mailgun');

async function testConnection() {
  try {
    // Test by listing domains
    const domains = await mg.domains.list();
    console.log('Connected! Domains:', domains.map(d => d.name));

    // Verify your domain exists
    const myDomain = await mg.domains.get(domain);
    console.log('Domain status:', myDomain.state);
  } catch (error) {
    console.error('Connection failed:', error.message);
    if (error.status === 401) {
      console.error('Check: API key and region match');
    }
  }
}

testConnection();
```

Run: `node test-mailgun.js`
</verification>
