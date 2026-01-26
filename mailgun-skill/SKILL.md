---
name: mailgun-skill
description: Integrate Mailgun email delivery into applications. Full lifecycle - setup, send, webhooks, debug, optimize. Covers Node.js/JavaScript and Laravel/PHP stacks. Includes best practices, anti-patterns, deliverability strategies, use cases, and tips.
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

## What framework are you using?

1. **Laravel** (PHP)
2. **Node.js** (JavaScript/TypeScript)

**After framework selection, what would you like to do?**

**Implementation:**
1. **Set up Mailgun** - Install packages, configure environment, verify domain
2. **Send emails** - Transactional, templates, bulk sending
3. **Handle webhooks** - Process bounces, complaints, events
4. **Debug delivery issues** - Troubleshoot emails not arriving
5. **Optimize deliverability** - Improve inbox placement

**Knowledge & Strategy:**
6. **Best practices** - Recommended patterns and approaches
7. **Use cases** - Common scenarios (transactional, marketing, notifications)
8. **Anti-patterns** - What NOT to do (learn from common mistakes)
9. **Tips and tricks** - Shortcuts, debugging tricks, dashboard tips

**Wait for response before proceeding.**
</intake>

<routing>

## Framework Selection

| Response | Framework | References to Load |
|----------|-----------|-------------------|
| 1, "laravel", "php" | Laravel | `references/laravel/setup.md` + `references/laravel/patterns.md` |
| 2, "node", "nodejs", "javascript", "typescript", "js", "ts" | Node.js | `references/nodejs/setup.md` + `references/nodejs/patterns.md` |

## Task Routing

**Implementation workflows:**

| Response | Workflow |
|----------|----------|
| 1, "setup", "configure", "install", "new" | `workflows/setup-mailgun.md` |
| 2, "send", "email", "template", "bulk" | `workflows/send-emails.md` |
| 3, "webhook", "bounce", "complaint", "event" | `workflows/handle-webhooks.md` |
| 4, "debug", "not delivered", "spam", "issue" | `workflows/debug-delivery.md` |
| 5, "optimize", "deliverability", "reputation" | `workflows/optimize-deliverability.md` |

**Knowledge references (read directly, no workflow):**

| Response | Reference |
|----------|-----------|
| 6, "best practice", "recommended", "good patterns" | `references/core/best-practices.md` |
| 7, "use case", "scenario", "example", "how to implement" | `references/core/use-cases.md` |
| 8, "anti-pattern", "bad", "avoid", "don't", "mistake" | `references/core/anti-patterns.md` |
| 9, "tips", "tricks", "shortcuts" | `references/core/tips-and-tricks.md` |

**After identifying framework and task:**
1. For implementation tasks: Read framework-specific references, then workflow file
2. For knowledge tasks: Read the reference file directly
3. Adapt workflow patterns to the selected framework

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

### Core (Framework-Agnostic)
All in `references/core/`:

**Implementation:**
- dns-configuration.md - SPF, DKIM, DMARC setup
- api-keys-security.md - Key management and security
- webhooks-events.md - Webhook payloads and event types
- templates-personalization.md - Handlebars templates and variables
- suppressions.md - Bounces, complaints, unsubscribes
- deliverability.md - Reputation and inbox placement
- rate-limits.md - API and sending limits
- error-codes.md - API and SMTP error reference
- common-issues.md - Troubleshooting guide

**Strategy & Knowledge:**
- best-practices.md - Recommended patterns and approaches
- anti-patterns.md - What NOT to do
- use-cases.md - Common scenarios (transactional, marketing, notifications)
- tips-and-tricks.md - Shortcuts, debugging, dashboard tips

### Laravel
All in `references/laravel/`:
- setup.md - Installation and configuration
- patterns.md - Mailables, queuing, webhooks, notifications

### Node.js
All in `references/nodejs/`:
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

<context7_usage>

## Getting Current Documentation

**Mailgun Official:**
```
mcp__context7__resolve-library-id:
  libraryName: "mailgun"
  query: "your question"
```

**Laravel Mail:**
```
mcp__context7__query-docs:
  libraryId: "/laravel/docs"
  query: "mail mailgun configuration"
```

**Node.js Mailgun SDK:**
```
mcp__context7__resolve-library-id:
  libraryName: "mailgun.js"
  query: "your question"
```

</context7_usage>

<quick_reference>

## Package Installation

| Framework | Command |
|-----------|---------|
| Laravel | Built-in (configure `config/mail.php`) |
| Node.js | `npm install mailgun.js form-data` |

## Environment Variables

| Variable | Laravel | Node.js |
|----------|---------|---------|
| API Key | `MAILGUN_SECRET` | `MAILGUN_API_KEY` |
| Domain | `MAILGUN_DOMAIN` | `MAILGUN_DOMAIN` |
| Region | `MAILGUN_ENDPOINT` (api.eu.mailgun.net for EU) | `MAILGUN_REGION` (eu or us) |
| From Address | `MAIL_FROM_ADDRESS` | Set per message |
| From Name | `MAIL_FROM_NAME` | Set per message |

## API Endpoints

| Region | Base URL |
|--------|----------|
| US | `https://api.mailgun.net/v3` |
| EU | `https://api.eu.mailgun.net/v3` |

## Common Operations

```bash
# Send test email (curl)
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/YOUR_DOMAIN/messages \
  -F from='Test <test@YOUR_DOMAIN>' \
  -F to='recipient@example.com' \
  -F subject='Test' \
  -F text='Hello'

# Check domain verification
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/domains/YOUR_DOMAIN

# List recent events
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/YOUR_DOMAIN/events
```

## Webhook Events

| Event | When | Action |
|-------|------|--------|
| `delivered` | Email reached inbox | Log success |
| `opened` | Recipient opened email | Track engagement |
| `clicked` | Recipient clicked link | Track engagement |
| `bounced` | Permanent failure | Remove from list |
| `complained` | Marked as spam | Remove immediately |
| `unsubscribed` | User unsubscribed | Remove from list |

</quick_reference>
