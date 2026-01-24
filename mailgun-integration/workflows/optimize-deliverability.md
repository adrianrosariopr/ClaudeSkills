# Workflow: Optimize Deliverability

<required_reading>
**Read these reference files NOW:**
1. references/deliverability.md
2. references/dns-configuration.md
3. references/suppressions.md
</required_reading>

<process>
## Step 1: Complete DNS Authentication

### SPF (Sender Policy Framework)

Verifies you're authorized to send from your domain.

```
v=spf1 include:mailgun.org ~all
```

**Check:** `dig TXT yourdomain.com | grep spf`

**Common issue:** SPF has 10 DNS lookup limit. Too many `include:` statements breaks it.

### DKIM (DomainKeys Identified Mail)

Cryptographically signs your emails.

Add the TXT record Mailgun provides (long key string).

**Check:** `dig TXT selector._domainkey.yourdomain.com`

### DMARC (Domain-based Message Authentication)

Tells receivers how to handle failed SPF/DKIM.

```
v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com
```

Start with `p=none` (monitoring), then progress to `p=quarantine` or `p=reject`.

**Impact:** Gmail and Yahoo require DMARC for bulk senders (2024+ requirement).

## Step 2: Warm Up New Domains/IPs

Don't send thousands of emails immediately on a new domain.

**Warm-up schedule:**
| Day | Volume |
|-----|--------|
| 1-2 | 50-100 |
| 3-4 | 200-500 |
| 5-7 | 500-1,000 |
| Week 2 | 1,000-5,000 |
| Week 3 | 5,000-10,000 |
| Week 4+ | Full volume |

**Mailgun feature:** Enable automated IP warm-up in dashboard.

**During warm-up:**
- Send to your most engaged users first
- Monitor bounce rates closely
- Pause if bounces exceed 2%

## Step 3: Maintain List Hygiene

### Remove Invalid Addresses

Use Mailgun's Email Validation API:

```javascript
// Node.js
const validation = await mg.validate.get('user@example.com');
if (validation.risk === 'high' || !validation.is_valid) {
  // Don't send to this address
}
```

### Process Bounces and Complaints

Set up webhooks (see `workflows/handle-webhooks.md`) to automatically:
- Mark bounced addresses as invalid
- Unsubscribe complainers
- Update your database

### Regular List Cleaning

Monthly:
1. Export your suppression list from Mailgun
2. Remove those addresses from your database
3. Re-validate old addresses (>6 months inactive)

## Step 4: Optimize Email Content

### Subject Lines
- Avoid ALL CAPS
- Avoid excessive punctuation (!!!, ???)
- Avoid spam trigger words: "FREE", "ACT NOW", "LIMITED TIME"
- Keep under 50 characters

### HTML Content
- Balance text-to-image ratio (more text, fewer images)
- Include plain text version
- Use inline CSS (not external stylesheets)
- Avoid JavaScript (it's stripped)
- Include unsubscribe link

### Structure
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
</head>
<body style="font-family: Arial, sans-serif; margin: 0; padding: 20px;">
  <!-- Preheader text (shows in inbox preview) -->
  <span style="display: none;">Preview text here...</span>

  <table width="100%" cellpadding="0" cellspacing="0">
    <tr>
      <td align="center">
        <table width="600" cellpadding="0" cellspacing="0">
          <!-- Email content -->
        </table>
      </td>
    </tr>
  </table>

  <p style="font-size: 12px; color: #666;">
    <a href="%unsubscribe_url%">Unsubscribe</a>
  </p>
</body>
</html>
```

## Step 5: Implement Engagement Tracking

Track metrics to identify and fix issues early:

```javascript
// Calculate engagement metrics
function calculateMetrics(stats) {
  return {
    deliveryRate: (stats.delivered / stats.sent) * 100,
    openRate: (stats.opened / stats.delivered) * 100,
    clickRate: (stats.clicked / stats.delivered) * 100,
    bounceRate: (stats.bounced / stats.sent) * 100,
    complaintRate: (stats.complained / stats.delivered) * 100
  };
}
```

**Target benchmarks:**
| Metric | Good | Warning | Bad |
|--------|------|---------|-----|
| Delivery rate | >98% | 95-98% | <95% |
| Open rate | >20% | 10-20% | <10% |
| Bounce rate | <1% | 1-2% | >2% |
| Complaint rate | <0.05% | 0.05-0.1% | >0.1% |

## Step 6: Segment Your Lists

Don't send the same email to everyone:

**By engagement:**
- Active (opened in last 30 days)
- Inactive (30-90 days)
- Dormant (90+ days)

**Strategy:**
- Send most emails to active users
- Re-engagement campaigns for inactive
- Sunset campaigns for dormant (then remove)

```javascript
// Example segmentation
const activeUsers = users.filter(u => u.lastOpened > daysAgo(30));
const inactiveUsers = users.filter(u =>
  u.lastOpened > daysAgo(90) && u.lastOpened <= daysAgo(30)
);
const dormantUsers = users.filter(u => u.lastOpened <= daysAgo(90));
```

## Step 7: Monitor Reputation Tools

### Google Postmaster Tools
1. Sign up: https://postmaster.google.com
2. Verify your domain
3. Monitor: spam rate, IP reputation, domain reputation

### Microsoft SNDS
1. Sign up: https://sendersupport.olc.protection.outlook.com/snds/
2. Track delivery to Outlook/Hotmail

### Sender Score
Check your sending IP reputation at: https://senderscore.org

## Step 8: Set Up Feedback Loops

Register for ISP feedback loops to receive spam complaints:

- **Yahoo:** https://help.yahoo.com/kb/postmaster
- **Microsoft:** https://sendersupport.olc.protection.outlook.com/snds/
- **AOL:** https://postmaster.aol.com/

Mailgun handles many FBLs automatically and sends complaints to your webhook.

## Step 9: Use Dedicated IPs (High Volume)

For 50,000+ emails/month, consider dedicated IP:
- Full control over reputation
- Not affected by other senders
- Requires proper warm-up

Contact Mailgun for dedicated IP options.
</process>

<success_criteria>
Deliverability is optimized when:
- [ ] SPF, DKIM, DMARC all configured and passing
- [ ] Delivery rate >98%
- [ ] Bounce rate <1%
- [ ] Complaint rate <0.05%
- [ ] Gmail Postmaster shows good reputation
- [ ] Emails consistently reach inbox (not spam)
- [ ] Webhooks handling bounces/complaints automatically
- [ ] Regular list hygiene process in place
</success_criteria>

<anti_patterns>
Avoid:
- Sending to purchased email lists
- Ignoring bounce/complaint data
- Skipping email warm-up
- Sending to dormant addresses
- Using generic "from" addresses (noreply@)
- Missing unsubscribe links in marketing emails
- Inconsistent sending patterns (spikes)
- Neglecting engagement metrics
</anti_patterns>
