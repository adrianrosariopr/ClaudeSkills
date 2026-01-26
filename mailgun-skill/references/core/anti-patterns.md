# Mailgun Anti-Patterns

## Domain Configuration

### Using Root Domain for Transactional Email
```
Bad:  yourapp.com
Good: mail.yourapp.com
```

**Why it's bad:**
- Damages your main domain reputation if deliverability issues occur
- Conflicts with existing MX records (breaks receiving email)
- Can't isolate transactional vs marketing reputation
- Recovery from blacklisting affects entire domain

### Skipping DNS Verification
Sending emails before completing SPF/DKIM/DMARC setup:
- Emails land in spam folders
- Low engagement damages reputation further
- May get domain blacklisted quickly

**Fix:** Wait for all DNS checks to pass before sending any production email.

### Sharing Sending Domain Across Unrelated Applications
Using one domain for multiple apps with different purposes:
```
Bad:  mail.yourcompany.com for AppA, AppB, and Marketing
Good: notifications.appa.com, alerts.appb.com, marketing.yourcompany.com
```

**Why it's bad:** One app's poor practices affects all others.

---

## Sending Patterns

### Sending Email Synchronously in Web Requests
```php
// BAD - Blocks request, user waits
public function register(Request $request) {
    $user = User::create($request->all());
    Mail::to($user)->send(new WelcomeEmail($user)); // Slow!
    return redirect('/dashboard');
}

// GOOD - Queue it
public function register(Request $request) {
    $user = User::create($request->all());
    Mail::to($user)->queue(new WelcomeEmail($user)); // Fast!
    return redirect('/dashboard');
}
```

**Why it's bad:**
- Slow user experience (email APIs take 100-500ms)
- Request fails if Mailgun is down
- Can't retry failed sends

### Sending Individual API Calls for Bulk Email
```javascript
// BAD - 1000 API calls, slow, rate limited
for (const user of users) {
  await mailgun.messages.create(domain, {
    to: user.email,
    subject: 'Newsletter',
    html: newsletter
  });
}

// GOOD - Single batch call
await mailgun.messages.create(domain, {
  to: users.map(u => u.email),
  subject: 'Newsletter',
  html: newsletter,
  'recipient-variables': JSON.stringify(
    users.reduce((acc, u) => ({ ...acc, [u.email]: { name: u.name } }), {})
  )
});
```

**Why it's bad:**
- Hits rate limits quickly
- Extremely slow for large lists
- More failure points

### Not Implementing Unsubscribe
```html
<!-- BAD - No way to unsubscribe -->
<p>Thanks for reading!</p>

<!-- GOOD - Clear unsubscribe link -->
<p>
  <a href="%unsubscribe_url%">Unsubscribe</a> |
  <a href="%mailing_list_unsubscribe_url%">Manage preferences</a>
</p>
```

**Why it's bad:**
- Violates CAN-SPAM, GDPR, CASL laws
- Users mark as spam instead (hurts reputation)
- Can result in Mailgun account suspension

### Sending to Purchased/Scraped Lists
Never send to addresses you didn't collect yourself with explicit consent.

**Why it's bad:**
- High bounce rates (many invalid addresses)
- High complaint rates (people didn't sign up)
- Reputation destruction
- Account suspension
- Legal liability

---

## Webhook Handling

### Not Verifying Webhook Signatures
```javascript
// BAD - Anyone can spoof this
app.post('/webhooks/mailgun', (req, res) => {
  processEvent(req.body); // Dangerous!
  res.send('OK');
});

// GOOD - Verify signature
app.post('/webhooks/mailgun', (req, res) => {
  if (!verifySignature(req.body)) {
    return res.status(401).send('Unauthorized');
  }
  processEvent(req.body);
  res.send('OK');
});
```

**Why it's bad:**
- Attackers can trigger fake events
- Could mark valid emails as bounced
- Could inject malicious data

### Ignoring Bounce/Complaint Webhooks
```javascript
// BAD - Only handling delivered events
webhookHandlers = {
  delivered: (event) => logSuccess(event)
  // Missing: bounced, complained, unsubscribed
};

// GOOD - Handle all important events
webhookHandlers = {
  delivered: (event) => logSuccess(event),
  bounced: (event) => markEmailInvalid(event),
  complained: (event) => unsubscribeImmediately(event),
  unsubscribed: (event) => honorPreference(event)
};
```

**Why it's bad:**
- Continues sending to dead addresses (damages reputation)
- Continues sending to complainers (more complaints)
- Violates user preferences (legal issues)

### Slow Webhook Processing
```javascript
// BAD - Mailgun may timeout waiting
app.post('/webhooks/mailgun', async (req, res) => {
  await updateDatabase(req.body);     // Slow
  await notifySlack(req.body);        // Slow
  await updateAnalytics(req.body);    // Slow
  res.send('OK'); // Too late, Mailgun gave up
});

// GOOD - Acknowledge fast, process later
app.post('/webhooks/mailgun', async (req, res) => {
  res.send('OK'); // Immediate response
  await queue.add('process-webhook', req.body);
});
```

**Why it's bad:**
- Mailgun retries on timeout, duplicate processing
- Can lose webhook events entirely
- Webhook endpoint appears unreliable

---

## Security Anti-Patterns

### Hardcoding API Keys
```javascript
// BAD
const mailgun = new Mailgun({
  key: 'key-abc123xyz789'  // Visible in source code!
});

// GOOD
const mailgun = new Mailgun({
  key: process.env.MAILGUN_API_KEY
});
```

**Why it's bad:**
- Keys in source control get leaked
- Keys in client-side code are public
- Can't rotate without code changes

### Logging API Keys
```javascript
// BAD
console.log('Initializing Mailgun with key:', apiKey);
logger.info({ config: { apiKey, domain } });

// GOOD
console.log('Initializing Mailgun with key:', apiKey.slice(0, 8) + '...');
logger.info({ config: { domain } }); // Omit key entirely
```

### Using Main API Key for Sending
```
Bad:  Using your main account API key in application code
Good: Creating domain-specific SMTP credentials or sending keys
```

**Why it's bad:**
- Main key has full account access (billing, DNS, etc.)
- If compromised, entire account is at risk
- Can't revoke without affecting everything

---

## Deliverability Anti-Patterns

### Blasting High Volume from New Domain
```
Day 1: Send 50,000 emails
Result: Domain blacklisted, emails blocked
```

**Fix:** Warm up gradually:
- Week 1: 50-100/day
- Week 2: 500/day
- Week 3: 2000/day
- Week 4+: Increase 20-30% weekly

### Ignoring Engagement Metrics
Not monitoring or acting on:
- Low open rates (<10%)
- High bounce rates (>5%)
- Complaint rates (>0.1%)

**Fix:** Set up alerts and automated responses:
- Pause sending if complaint rate spikes
- Clean list when bounce rate rises
- Segment inactive users for re-engagement

### Using No-Reply Addresses
```
Bad:  noreply@yourapp.com
Good: support@yourapp.com, hello@yourapp.com
```

**Why it's bad:**
- Users can't reply (poor experience)
- Replies to noreply bounce (hurts reputation)
- Some spam filters penalize noreply addresses

### Sending Same Content to Everyone
```javascript
// BAD - Generic blast
await sendEmail({
  subject: 'Newsletter',
  to: allUsers,
  html: genericContent
});

// GOOD - Personalized
await sendEmail({
  subject: `${user.name}, your personalized update`,
  to: user.email,
  html: renderTemplate(userSpecificContent)
});
```

**Why it's bad:**
- Lower engagement = worse reputation
- Higher unsubscribe/complaint rates
- Looks like spam

---

## Error Handling Anti-Patterns

### Swallowing Errors
```javascript
// BAD
try {
  await mailgun.messages.create(domain, emailData);
} catch (e) {
  // Silent failure, email lost
}

// GOOD
try {
  await mailgun.messages.create(domain, emailData);
} catch (e) {
  logger.error('Email failed', { error: e, emailData });
  await retryQueue.add(emailData);
}
```

### Not Handling Rate Limits
```javascript
// BAD - Crashes on 429
const result = await mailgun.messages.create(domain, data);

// GOOD - Retry with backoff
const result = await sendWithRetry(data);
```

### Retrying Non-Retryable Errors
```javascript
// BAD - Retrying invalid address errors
while (retries < 5) {
  try {
    await mailgun.messages.create(domain, data);
    break;
  } catch (e) {
    retries++; // Will fail forever if address invalid
  }
}

// GOOD - Only retry transient errors
if (error.status === 429 || error.status >= 500) {
  await retry();
} else {
  throw error; // Don't retry 400-level errors
}
```
