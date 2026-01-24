<overview>
Mailgun maintains suppression lists to protect sender reputation. Emails to suppressed addresses are automatically dropped.
</overview>

<suppression_types>
## Suppression Types

<bounces>
**Bounces**
- Addresses that permanently failed delivery
- Added automatically when 5xx SMTP error received
- Prevents wasting resources on invalid addresses
</bounces>

<complaints>
**Complaints**
- Recipients who marked email as spam
- Added via feedback loops from ISPs
- Critical to honor - continued sending damages reputation
</complaints>

<unsubscribes>
**Unsubscribes**
- Recipients who clicked unsubscribe link
- Added when using Mailgun's unsubscribe handling
- Required to honor for CAN-SPAM/GDPR compliance
</unsubscribes>
</suppression_types>

<how_suppressions_work>
## How Suppressions Work

When you try to send to a suppressed address:
1. Mailgun checks suppression lists
2. If found, message is **dropped** (not sent)
3. A `permanent_fail` event is generated with code 605/606/607
4. No delivery attempt is made

**Suppression codes:**
- 605: Previously bounced
- 606: Previously complained (spam)
- 607: Previously unsubscribed
</how_suppressions_work>

<viewing_suppressions>
## Viewing Suppressions

<via_dashboard>
**Via Dashboard:**
1. Go to **Sending → Suppressions**
2. Select tab: Bounces, Complaints, or Unsubscribes
3. Search by email address
4. Export as CSV if needed
</via_dashboard>

<via_api>
**Via API (Node.js):**
```javascript
// List bounces
const bounces = await mg.suppressions.list(domain, 'bounces', {
  limit: 100
});

// List complaints
const complaints = await mg.suppressions.list(domain, 'complaints');

// List unsubscribes
const unsubscribes = await mg.suppressions.list(domain, 'unsubscribes');

// Check specific address
const bounce = await mg.suppressions.get(domain, 'bounces', 'user@example.com');
```
</via_api>
</viewing_suppressions>

<managing_suppressions>
## Managing Suppressions

<adding>
**Adding to suppression list:**
```javascript
// Add bounce
await mg.suppressions.create(domain, 'bounces', {
  address: 'bad@example.com',
  code: 550,
  error: 'Mailbox does not exist'
});

// Add complaint
await mg.suppressions.create(domain, 'complaints', {
  address: 'complained@example.com'
});

// Add unsubscribe
await mg.suppressions.create(domain, 'unsubscribes', {
  address: 'user@example.com',
  tag: '*'  // Unsubscribe from all tags, or specific tag
});
```
</adding>

<removing>
**Removing from suppression list:**
```javascript
// Remove bounce (use cautiously)
await mg.suppressions.destroy(domain, 'bounces', 'user@example.com');

// Remove unsubscribe (only if user re-subscribes)
await mg.suppressions.destroy(domain, 'unsubscribes', 'user@example.com');
```

**Warning:** Removing from complaint list is rarely appropriate. The recipient explicitly marked you as spam.
</removing>

<bulk_operations>
**Bulk operations:**
```javascript
// Add multiple bounces
const bounces = [
  { address: 'bad1@example.com', code: 550 },
  { address: 'bad2@example.com', code: 550 }
];

for (const bounce of bounces) {
  await mg.suppressions.create(domain, 'bounces', bounce);
}
```
</bulk_operations>
</managing_suppressions>

<syncing_with_database>
## Syncing with Your Database

<webhook_approach>
**Recommended: Use webhooks**

Set up webhooks to automatically update your database:

```javascript
// Webhook handler
app.post('/webhooks/mailgun', async (req, res) => {
  const { 'event-data': eventData } = req.body;
  const email = eventData.recipient;

  switch (eventData.event) {
    case 'failed':
      if (eventData.severity === 'permanent') {
        await db.users.update({ email }, {
          emailStatus: 'bounced',
          canEmail: false
        });
      }
      break;

    case 'complained':
      await db.users.update({ email }, {
        emailStatus: 'complained',
        canEmail: false,
        subscribed: false
      });
      break;

    case 'unsubscribed':
      await db.users.update({ email }, {
        subscribed: false
      });
      break;
  }

  res.status(200).json({ received: true });
});
```
</webhook_approach>

<periodic_sync>
**Alternative: Periodic sync**

Export suppressions and update database:

```javascript
async function syncSuppressions() {
  // Get all bounces
  const bounces = await mg.suppressions.list(domain, 'bounces', { limit: 1000 });

  for (const bounce of bounces.items) {
    await db.users.update(
      { email: bounce.address },
      { emailStatus: 'bounced', canEmail: false }
    );
  }

  // Repeat for complaints and unsubscribes
}
```
</periodic_sync>
</syncing_with_database>

<checking_before_send>
## Checking Before Sending

<pre_send_check>
**Option 1: Check suppression before sending**

```javascript
async function canSendTo(email) {
  try {
    await mg.suppressions.get(domain, 'bounces', email);
    return false; // Found in bounces
  } catch (e) {
    if (e.status === 404) {
      // Not in bounces, check complaints
      try {
        await mg.suppressions.get(domain, 'complaints', email);
        return false; // Found in complaints
      } catch (e2) {
        if (e2.status === 404) {
          return true; // Not suppressed
        }
      }
    }
  }
  return false;
}
```
</pre_send_check>

<local_tracking>
**Option 2: Track locally (recommended)**

More efficient than API calls:

```javascript
// In your User model/service
async function canSendTo(userId) {
  const user = await db.users.findById(userId);
  return user.canEmail && user.subscribed;
}

// Update via webhooks (see above)
```
</local_tracking>
</checking_before_send>

<best_practices>
## Best Practices

<do>
**Do:**
- Set up webhooks to handle bounces/complaints automatically
- Update your database when suppressions occur
- Check your local canEmail flag before sending
- Export and review suppressions periodically
- Honor unsubscribes immediately
</do>

<dont>
**Don't:**
- Ignore suppression events
- Remove complaints without user's explicit re-consent
- Continue sending to bounced addresses
- Manually remove bounces without verification
- Import purchased lists (high bounce/complaint rate)
</dont>
</best_practices>
