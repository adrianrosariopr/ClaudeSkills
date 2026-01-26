<overview>
Email deliverability is the ability to reach recipients' inboxes (not spam). It depends on technical configuration, sender reputation, and content quality.
</overview>

<deliverability_factors>
## Key Factors

<authentication>
**1. Email Authentication**
- SPF: Authorizes sending servers
- DKIM: Cryptographically signs emails
- DMARC: Policy for handling failures

All three should be configured. See dns-configuration.md.
</authentication>

<reputation>
**2. Sender Reputation**
- IP reputation (shared or dedicated IP)
- Domain reputation
- Historical bounce/complaint rates
- Engagement metrics (opens, clicks)
</reputation>

<content>
**3. Email Content**
- Subject line quality
- Text-to-image ratio
- Spam trigger words
- Unsubscribe mechanism
</content>

<list_quality>
**4. List Quality**
- Valid email addresses
- Engaged subscribers
- Regular list cleaning
- Permission-based collection
</list_quality>
</deliverability_factors>

<metrics_to_monitor>
## Metrics to Monitor

| Metric | Good | Warning | Action Needed |
|--------|------|---------|---------------|
| Delivery Rate | >98% | 95-98% | <95% |
| Bounce Rate | <1% | 1-2% | >2% |
| Complaint Rate | <0.05% | 0.05-0.1% | >0.1% |
| Open Rate | >20% | 10-20% | <10% |
| Click Rate | >2% | 1-2% | <1% |

<calculating>
```javascript
function calculateMetrics(stats) {
  return {
    deliveryRate: (stats.delivered / stats.sent * 100).toFixed(2),
    bounceRate: (stats.bounced / stats.sent * 100).toFixed(2),
    complaintRate: (stats.complaints / stats.delivered * 100).toFixed(3),
    openRate: (stats.opens / stats.delivered * 100).toFixed(2),
    clickRate: (stats.clicks / stats.delivered * 100).toFixed(2)
  };
}
```
</calculating>
</metrics_to_monitor>

<ip_warming>
## IP Warming

New IPs have no reputation. Build it gradually:

<warm_up_schedule>
| Week | Daily Volume |
|------|-------------|
| 1 | 50-200 |
| 2 | 200-500 |
| 3 | 500-1,000 |
| 4 | 1,000-5,000 |
| 5 | 5,000-10,000 |
| 6+ | Increase 50% weekly |
</warm_up_schedule>

<warming_tips>
**During warm-up:**
- Send to most engaged users first
- Maintain consistent daily volume
- Monitor bounces closely (pause if >2%)
- Don't send to cold lists
- Mailgun offers automated IP warm-up
</warming_tips>
</ip_warming>

<reputation_monitoring>
## Reputation Monitoring Tools

<google_postmaster>
**Google Postmaster Tools**
- URL: https://postmaster.google.com
- Monitor: Domain reputation, IP reputation, spam rate
- Required for Gmail deliverability insights
</google_postmaster>

<microsoft_snds>
**Microsoft SNDS**
- URL: https://sendersupport.olc.protection.outlook.com/snds/
- Monitor: Outlook/Hotmail delivery metrics
- See complaint rates and filter results
</microsoft_snds>

<sender_score>
**Sender Score**
- URL: https://senderscore.org
- Check IP reputation (0-100 scale)
- Score >80 is good, <70 needs attention
</sender_score>

<blacklist_check>
**Blacklist Check**
- URL: https://mxtoolbox.com/blacklists.aspx
- Check if IP/domain is blacklisted
- Common lists: Spamhaus, Barracuda, Spamcop
</blacklist_check>
</reputation_monitoring>

<content_best_practices>
## Content Best Practices

<subject_lines>
**Subject Lines:**
- Keep under 50 characters
- Avoid ALL CAPS
- Avoid excessive punctuation (!!!)
- Avoid spam triggers: "FREE", "ACT NOW", "$$$"
- Be specific and relevant
</subject_lines>

<email_body>
**Email Body:**
- Balance text and images (60/40 text-heavy)
- Include plain text version
- Avoid large images with little text
- No JavaScript (stripped by clients)
- Inline CSS only
</email_body>

<required_elements>
**Required Elements:**
- Physical mailing address (CAN-SPAM)
- Unsubscribe link (working, one-click)
- Sender identification
- Accurate subject line
</required_elements>
</content_best_practices>

<list_hygiene>
## List Hygiene

<regular_cleaning>
**Regular cleaning:**
1. Remove hard bounces immediately
2. Remove complainers immediately
3. Re-engage inactive subscribers (90+ days)
4. Sunset dormant addresses (180+ days)
5. Validate addresses periodically
</regular_cleaning>

<validation>
**Email validation:**
```javascript
// Use Mailgun validation API
const result = await mg.validate.get(email);

if (!result.is_valid || result.risk === 'high') {
  // Don't add to list
}

if (result.is_disposable) {
  // Consider rejecting disposable addresses
}
```
</validation>

<engagement_segmentation>
**Segment by engagement:**
```javascript
const segments = {
  active: users.filter(u => u.lastOpen > daysAgo(30)),
  inactive: users.filter(u =>
    u.lastOpen > daysAgo(90) && u.lastOpen <= daysAgo(30)
  ),
  dormant: users.filter(u => u.lastOpen <= daysAgo(90))
};

// Send most emails to active
// Re-engagement campaigns to inactive
// Sunset campaigns to dormant, then remove
```
</engagement_segmentation>
</list_hygiene>

<2024_requirements>
## 2024+ Industry Requirements

Gmail and Yahoo (2024), Microsoft (2025) require:

<authentication_required>
**Authentication:**
- SPF must pass
- DKIM must pass
- DMARC must be published
</authentication_required>

<one_click_unsubscribe>
**One-click unsubscribe:**
- List-Unsubscribe header required for bulk mail
- Must process within 2 days

```javascript
// Add headers
{
  'h:List-Unsubscribe': '<mailto:unsubscribe@yourdomain.com>, <https://yourdomain.com/unsubscribe?id=123>',
  'h:List-Unsubscribe-Post': 'List-Unsubscribe=One-Click'
}
```
</one_click_unsubscribe>

<spam_rate>
**Spam complaint rate:**
- Must stay below 0.1% (ideally <0.05%)
- Monitor via Google Postmaster Tools
</spam_rate>
</2024_requirements>
