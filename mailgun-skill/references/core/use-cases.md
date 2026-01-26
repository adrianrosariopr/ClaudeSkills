# Mailgun Use Cases

## Transactional Email

### Account Emails
Critical, time-sensitive emails triggered by user actions.

**Examples:**
- Welcome/confirmation emails
- Password reset
- Email verification
- Account security alerts
- Two-factor authentication codes

**Best Practices:**
- Send immediately (high priority queue)
- Simple, clear design
- No marketing content
- Track delivery for security compliance

```javascript
// Password reset - time-sensitive
await mailgun.messages.create(domain, {
  to: user.email,
  subject: 'Reset your password',
  template: 'password-reset',
  'o:tag': ['transactional', 'password-reset'],
  'h:X-Mailgun-Variables': JSON.stringify({
    reset_url: `https://app.com/reset?token=${token}`,
    expires_in: '1 hour'
  })
});
```

### Order/Purchase Confirmations
Triggered by e-commerce transactions.

**Examples:**
- Order confirmation
- Payment receipt
- Shipping notification
- Delivery confirmation
- Refund processed

**Best Practices:**
- Include order details inline (users often search for these)
- Attach PDF receipt if needed
- Include tracking links for shipping

```javascript
await mailgun.messages.create(domain, {
  to: customer.email,
  subject: `Order #${order.id} confirmed`,
  template: 'order-confirmation',
  'h:X-Mailgun-Variables': JSON.stringify({
    order_id: order.id,
    items: order.items,
    total: order.total,
    shipping_address: order.address
  })
});
```

### Activity Notifications
Real-time updates about actions in the application.

**Examples:**
- New comment on post
- Someone followed you
- File shared with you
- Payment received
- Subscription renewal

**Best Practices:**
- Provide digest options (daily/weekly instead of each event)
- Deep links to relevant page
- Clear unsubscribe per notification type

```javascript
await mailgun.messages.create(domain, {
  to: user.email,
  subject: `${commenter.name} commented on your post`,
  template: 'new-comment',
  'o:tag': ['notification', 'comment'],
  'h:X-Mailgun-Variables': JSON.stringify({
    commenter_name: commenter.name,
    comment_preview: comment.text.slice(0, 100),
    post_url: `https://app.com/posts/${post.id}`
  })
});
```

---

## Marketing Email

### Newsletter
Regular content updates to engaged subscribers.

**Best Practices:**
- Use Mailgun mailing lists for management
- Include preference center link
- A/B test subject lines
- Schedule for optimal times (test for your audience)
- Segment by engagement level

```javascript
// Send to mailing list with recipient variables
await mailgun.messages.create(domain, {
  to: 'newsletter@mail.yourapp.com', // Mailing list
  subject: 'This week at YourApp',
  template: 'weekly-newsletter',
  'o:tag': ['marketing', 'newsletter', 'weekly'],
  'o:tracking': true,
  'o:tracking-clicks': true,
  'o:tracking-opens': true
});
```

### Product Updates
Announce new features, improvements, or changes.

**Best Practices:**
- Segment by product usage (send relevant updates)
- Include clear value proposition
- Link to documentation/guides
- Time with feature launch

```javascript
await mailgun.messages.create(domain, {
  to: activeUsers.map(u => u.email),
  subject: 'New feature: Dark mode is here!',
  template: 'product-update',
  'recipient-variables': JSON.stringify(
    activeUsers.reduce((acc, u) => ({
      ...acc,
      [u.email]: { name: u.name, feature_usage: u.usageLevel }
    }), {})
  )
});
```

### Re-engagement Campaigns
Win back inactive users.

**Best Practices:**
- Define "inactive" (e.g., no login in 30 days)
- Offer incentive or highlight new features
- Series of 2-3 emails, then remove if no response
- Don't be desperate (tone matters)

```javascript
// Re-engagement sequence
const reEngagementSequence = [
  { day: 0, template: 'we-miss-you', subject: 'We miss you, %recipient.name%!' },
  { day: 7, template: 'whats-new', subject: 'See what\'s new at YourApp' },
  { day: 14, template: 'last-chance', subject: 'Before we say goodbye...' }
];

// After day 14 with no engagement, remove from active list
```

---

## Operational Email

### System Alerts
Automated notifications about system status.

**Examples:**
- Downtime notifications
- Scheduled maintenance
- Security incidents
- Usage limits approaching

**Best Practices:**
- High priority/immediate delivery
- Clear, actionable information
- Status page link
- Expected resolution time

```javascript
await mailgun.messages.create(domain, {
  to: 'admin-team@yourcompany.com',
  subject: '[URGENT] Database approaching storage limit',
  text: `Database is at 85% capacity. Action required within 24 hours.`,
  'o:tag': ['system', 'alert', 'urgent']
});
```

### Reports and Digests
Scheduled summary emails.

**Examples:**
- Daily/weekly analytics reports
- Activity summaries
- Invoice summaries
- Team activity digests

**Best Practices:**
- Consistent schedule
- Inline key metrics
- Link to full dashboard
- Option to adjust frequency

```javascript
await mailgun.messages.create(domain, {
  to: user.email,
  subject: `Your weekly report for ${weekRange}`,
  template: 'weekly-report',
  'h:X-Mailgun-Variables': JSON.stringify({
    metrics: {
      visitors: weeklyStats.visitors,
      conversions: weeklyStats.conversions,
      revenue: weeklyStats.revenue
    },
    dashboard_url: 'https://app.com/dashboard'
  })
});
```

### Billing and Invoices
Financial communication.

**Examples:**
- Invoice sent
- Payment due reminder
- Payment failed
- Subscription renewed
- Subscription expiring

**Best Practices:**
- Attach PDF invoice
- Clear payment instructions
- Retry instructions for failed payments
- Multiple reminders before cancellation

```javascript
await mailgun.messages.create(domain, {
  to: customer.email,
  subject: `Invoice #${invoice.number} - ${invoice.total}`,
  template: 'invoice',
  attachment: [invoicePdfBuffer],
  'h:X-Mailgun-Variables': JSON.stringify({
    invoice_number: invoice.number,
    amount: invoice.total,
    due_date: invoice.dueDate,
    payment_url: invoice.paymentUrl
  })
});
```

---

## Multi-Tenant/SaaS Patterns

### Customer-Branded Email
Send email on behalf of your customers.

**Setup:**
- Each customer has their own verified sending domain
- Or use subdomain per customer: `customer1.mail.yourapp.com`

```javascript
// Send as customer's brand
await mailgun.messages.create(customer.mailgunDomain, {
  from: `${customer.brandName} <noreply@${customer.mailgunDomain}>`,
  to: endUser.email,
  subject: 'Welcome to ' + customer.brandName,
  template: 'customer-welcome',
  'h:X-Mailgun-Variables': JSON.stringify({
    brand_name: customer.brandName,
    brand_logo: customer.logoUrl,
    brand_color: customer.primaryColor
  })
});
```

### Per-Tenant Webhook Routing
Route webhook events to appropriate tenant handlers.

```javascript
app.post('/webhooks/mailgun', async (req, res) => {
  const event = req.body['event-data'];
  const domain = event.envelope.sender.split('@')[1];
  const tenant = await Tenant.findByDomain(domain);

  // Route to tenant-specific handler
  await tenantWebhookHandlers[tenant.id].process(event);
  res.send('OK');
});
```

---

## Email Testing Patterns

### Staging Environment
Test email flows without sending to real users.

**Options:**
1. **Mailgun sandbox** - Free, limited
2. **Intercept in code** - Catch all emails in dev/staging
3. **Mailtrap/Mailhog** - Capture and inspect

```javascript
// Environment-based routing
const getMailer = () => {
  if (process.env.NODE_ENV === 'production') {
    return productionMailgun;
  }
  return sandboxMailgun; // Or redirect to Mailtrap
};
```

### Email Preview System
Let users preview emails before sending.

```javascript
// Generate preview without sending
app.get('/admin/email-preview/:template', async (req, res) => {
  const html = await renderTemplate(req.params.template, sampleData);
  res.send(html); // Display in browser
});
```

---

## High-Volume Patterns

### Rate Limiting Your Own Sends
Avoid overwhelming Mailgun or triggering spam filters.

```javascript
const Bottleneck = require('bottleneck');

const limiter = new Bottleneck({
  minTime: 100, // 10 emails per second max
  maxConcurrent: 5
});

const sendEmail = limiter.wrap(async (data) => {
  return mailgun.messages.create(domain, data);
});
```

### Priority Queues
Ensure critical emails go out first.

```javascript
// Separate queues by priority
const highPriorityQueue = new Queue('email-high');   // Password reset, 2FA
const normalQueue = new Queue('email-normal');        // Notifications
const lowPriorityQueue = new Queue('email-low');     // Marketing

// Worker processes high priority first
const processQueues = async () => {
  await highPriorityQueue.process(sendEmail);
  await normalQueue.process(sendEmail);
  await lowPriorityQueue.process(sendEmail);
};
```

### Failure Recovery
Handle partial failures in bulk sends.

```javascript
const results = await Promise.allSettled(
  batches.map(batch => mailgun.messages.create(domain, batch))
);

const failed = results
  .filter(r => r.status === 'rejected')
  .map(r => r.reason);

if (failed.length > 0) {
  await retryQueue.addBulk(failed);
  logger.warn(`${failed.length} batches queued for retry`);
}
```
