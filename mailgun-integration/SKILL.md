---
name: mailgun-integration
description: Integrate Mailgun email delivery into applications. Full lifecycle - setup, send, webhooks, debug, optimize. Covers Node.js/JavaScript and Laravel/PHP stacks.
---

<essential_principles>
## How Mailgun Integration Works

Mailgun provides email delivery via REST API (recommended) or SMTP. The API approach is faster, scales better, and provides better debugging.

### 1. Domain Authentication is Critical

Before sending production emails, configure DNS records:
- **SPF** - Authorizes Mailgun to send on your behalf
- **DKIM** - Signs emails cryptographically to prevent tampering
- **DMARC** - Tells receivers how to handle failed authentication

Without proper DNS setup, emails land in spam. Domain verification typically takes 24-48 hours.

### 2. Use Subdomains for Sending

Never use your root domain for transactional email. Use a subdomain like `mail.yourdomain.com` or `notifications.yourdomain.com`. This:
- Protects your main domain reputation
- Prevents conflicts with existing MX records
- Allows separate reputation tracking

### 3. Handle Failures Gracefully

Mailgun categorizes failures as:
- **Temporary (soft bounce)** - Mailgun retries automatically
- **Permanent (hard bounce)** - Address doesn't exist, stop sending
- **Complaints** - User marked as spam, stop sending immediately

Always set up webhooks to handle bounces and complaints. Remove failed addresses from your lists.

### 4. Never Hardcode API Keys

Store credentials in environment variables:
- `MAILGUN_API_KEY` - Your private API key
- `MAILGUN_DOMAIN` - Your sending domain
- `MAILGUN_REGION` - `us` or `eu` (affects API endpoint)
</essential_principles>

<intake>
What would you like to do?

1. Set up Mailgun (new integration)
2. Send emails (transactional, templates, bulk)
3. Handle webhooks (bounces, complaints, events)
4. Debug delivery issues
5. Optimize deliverability
6. Something else

**Specify your stack:** Node.js/JavaScript or Laravel/PHP (or both)
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "setup", "configure", "install", "new" | `workflows/setup-mailgun.md` |
| 2, "send", "email", "template", "bulk" | `workflows/send-emails.md` |
| 3, "webhook", "bounce", "complaint", "event" | `workflows/handle-webhooks.md` |
| 4, "debug", "not delivered", "spam", "issue" | `workflows/debug-delivery.md` |
| 5, "optimize", "deliverability", "reputation" | `workflows/optimize-deliverability.md` |
| 6, other | Clarify intent, then route appropriately |

**Stack routing:**
- If user specifies Node.js/JS → emphasize Node.js examples
- If user specifies Laravel/PHP → emphasize Laravel examples
- If both or unspecified → show both approaches
</routing>

<verification_loop>
## After Every Change

1. **Test email delivery:**
```bash
# Node.js - run test script
node test-email.js

# Laravel - use tinker or artisan
php artisan tinker
# Mail::raw('Test', fn($m) => $m->to('test@example.com'));
```

2. **Check Mailgun logs:**
   - Dashboard → Sending → Logs
   - Look for: Accepted → Delivered (success) or Failed (investigate)

3. **Verify DNS (if setup):**
   - Dashboard → Sending → Domains → Verify DNS Settings
   - All checks should show green

Report to user:
- "Email sent: ✓ (message ID: xxx)"
- "Delivery status: Accepted/Delivered/Failed"
- "DNS verification: X of Y records verified"
</verification_loop>

<reference_index>
## Domain Knowledge

**Core (framework-agnostic)** in `references/core/`:
- dns-configuration.md - SPF, DKIM, DMARC setup
- api-keys-security.md - Key management and security
- webhooks-events.md - Webhook payloads and event types
- templates-personalization.md - Handlebars templates and variables
- suppressions.md - Bounces, complaints, unsubscribes
- deliverability.md - Reputation and inbox placement
- rate-limits.md - API and sending limits
- error-codes.md - API and SMTP error reference
- common-issues.md - Troubleshooting guide

**Laravel** in `references/laravel/`:
- setup.md - Installation and configuration
- patterns.md - Mailables, queuing, webhooks, notifications

**Node.js** in `references/nodejs/`:
- setup.md - Installation and initialization
- patterns.md - Sending, webhooks, API usage
</reference_index>

<workflows_index>
## Workflows

All in `workflows/`:

| File | Purpose |
|------|---------|
| setup-mailgun.md | Initial Mailgun setup and configuration |
| send-emails.md | Send transactional, template, and bulk emails |
| handle-webhooks.md | Process bounces, complaints, and events |
| debug-delivery.md | Troubleshoot delivery issues |
| optimize-deliverability.md | Improve inbox placement and reputation |
</workflows_index>
