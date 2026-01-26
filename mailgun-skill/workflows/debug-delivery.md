# Workflow: Debug Delivery Issues

<required_reading>
**Read these reference files NOW:**
1. references/common-issues.md
2. references/error-codes.md
3. references/dns-configuration.md
</required_reading>

<process>
## Step 1: Check Mailgun Logs

1. Go to **Sending → Logs**
2. Search for the recipient email
3. Check the event status and any error messages

**Status meanings:**
- **Accepted** → Mailgun received it, hasn't delivered yet
- **Delivered** → Successfully delivered to recipient server
- **Temporary Failed** → Soft bounce, Mailgun is retrying
- **Permanent Failed** → Hard bounce, delivery stopped

## Step 2: Diagnose by Status

### Accepted but Not Delivered

**Common causes:**
1. **DNS issues with recipient** - Check recipient's MX records:
   ```bash
   dig MX recipientdomain.com
   ```

2. **Scheduled for later** - Check if `o:deliverytime` was set

3. **Still in queue** - Wait and check again

4. **Recipient server is slow** - Some servers take time to respond

### Temporary Failure

**Common causes:**
1. **Mailbox full** - Recipient needs to clear space
2. **Server temporarily unavailable** - Will retry automatically
3. **Greylisting** - Recipient server delays first-time senders
4. **Rate limited** - You're sending too fast

**Action:** Wait for Mailgun to retry (up to 8 hours)

### Permanent Failure

**Check error code:**

| Code | Meaning | Action |
|------|---------|--------|
| 550 5.1.1 | Address doesn't exist | Remove from list |
| 550 5.7.1 | Blocked as spam | Check reputation, content |
| 552 | Mailbox full (permanent) | Remove from list |
| 553 | Invalid address format | Fix address format |
| 605 | Previously bounced | Address in suppression list |
| 607 | Previously complained | Address in suppression list |

### "Too Old" Status

Message exceeded retry period (usually 8 hours).

**Causes:**
- Recipient server consistently rejecting
- Network issues between Mailgun and recipient
- Poor sender reputation causing throttling

## Step 3: Check DNS Configuration

```bash
# Check SPF
dig TXT mail.yourdomain.com

# Check DKIM
dig TXT selector._domainkey.mail.yourdomain.com

# Check MX (if receiving)
dig MX mail.yourdomain.com
```

In Mailgun dashboard: **Sending → Domains → Verify DNS Settings**

**All checks should be green.** If not:
- Re-add the DNS records
- Wait 24-48 hours for propagation
- Ensure Cloudflare proxy is disabled for mail records

## Step 4: Check Suppression Lists

Emails to suppressed addresses are automatically dropped.

**Via Dashboard:**
1. Go to **Sending → Suppressions**
2. Search for the email address
3. Check Bounces, Complaints, and Unsubscribes tabs

**Via API (Node.js):**
```javascript
// Check if address is suppressed
const bounces = await mg.suppressions.list(domain, 'bounces', { address: 'user@example.com' });
const complaints = await mg.suppressions.list(domain, 'complaints', { address: 'user@example.com' });
const unsubscribes = await mg.suppressions.list(domain, 'unsubscribes', { address: 'user@example.com' });
```

**To remove from suppression (use cautiously):**
```javascript
await mg.suppressions.destroy(domain, 'bounces', 'user@example.com');
```

## Step 5: Check Sender Reputation

1. **Sender Score:** https://senderscore.org (enter your sending IP)
2. **Google Postmaster Tools:** For Gmail deliverability
3. **Microsoft SNDS:** For Outlook/Hotmail deliverability

**Signs of reputation issues:**
- High bounce rates (>2%)
- Spam complaints (>0.1%)
- Emails consistently going to spam
- Declining open rates

## Step 6: Test Email Content

Some email content triggers spam filters:

**Check for:**
- Spam trigger words ("FREE", "ACT NOW", "$$$$")
- Too many links
- Large images with little text
- Suspicious attachments
- Missing unsubscribe link (for marketing emails)
- HTML-only with no text version

**Test with:**
- https://www.mail-tester.com (send test email, get score)
- https://glockapps.com (inbox placement testing)

## Step 7: Verify API Response

### Node.js

```javascript
try {
  const result = await mg.messages.create(domain, messageData);
  console.log('Message ID:', result.id);
  console.log('Message:', result.message);
} catch (error) {
  console.error('Status:', error.status);
  console.error('Message:', error.message);
  console.error('Details:', error.details);
}
```

### Laravel

```php
try {
    Mail::to($user)->send(new MyEmail());
    Log::info('Email sent successfully');
} catch (\Exception $e) {
    Log::error('Email failed', [
        'message' => $e->getMessage(),
        'trace' => $e->getTraceAsString()
    ]);
}
```

**Common API errors:**

| Status | Meaning |
|--------|---------|
| 401 | Invalid API key or wrong region |
| 400 | Invalid request (check params) |
| 402 | Account disabled (billing issue) |
| 404 | Domain not found |
| 429 | Rate limited |

## Step 8: Check Rate Limits

**Account limits:**
- Free tier: 100 emails/day
- New accounts may have hourly limits

**API rate limits:**
- Check response headers for `X-RateLimit-*`
- Implement exponential backoff on 429 errors

```javascript
async function sendWithRetry(messageData, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await mg.messages.create(domain, messageData);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```
</process>

<success_criteria>
Delivery issue is resolved when:
- [ ] Root cause identified
- [ ] DNS issues fixed (if applicable)
- [ ] Suppression list cleaned up (if applicable)
- [ ] Content issues addressed (if applicable)
- [ ] Test email delivers successfully
- [ ] Logs show "Delivered" status
</success_criteria>

<diagnostic_checklist>
Quick checklist for any delivery issue:

1. [ ] Check Mailgun logs - what's the status?
2. [ ] DNS verified? All green checkmarks?
3. [ ] Is address on suppression list?
4. [ ] API returning success response?
5. [ ] Is account in good standing (not disabled)?
6. [ ] Rate limits exceeded?
7. [ ] Correct region configured (US vs EU)?
8. [ ] Email content clean (no spam triggers)?
</diagnostic_checklist>
