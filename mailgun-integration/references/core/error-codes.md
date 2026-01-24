<overview>
Reference for Mailgun API errors, SMTP error codes, and delivery status codes.
</overview>

<api_errors>
## API HTTP Status Codes

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | Success | Request completed |
| 400 | Bad Request | Check request parameters |
| 401 | Unauthorized | Invalid API key or wrong region |
| 402 | Payment Required | Account disabled, billing issue |
| 404 | Not Found | Domain or resource doesn't exist |
| 405 | Method Not Allowed | Wrong HTTP method |
| 413 | Payload Too Large | Attachment too big (>25MB) |
| 429 | Too Many Requests | Rate limited, implement backoff |
| 500 | Server Error | Mailgun issue, retry later |
| 502 | Bad Gateway | Mailgun issue, retry later |
| 503 | Service Unavailable | Mailgun issue, retry later |
</api_errors>

<delivery_status_codes>
## Delivery Status Codes

<mailgun_internal>
**Mailgun Internal Codes:**

| Code | Meaning | Action |
|------|---------|--------|
| 605 | Previously bounced | Address on bounce suppression list |
| 606 | Previously complained | Address on complaint suppression list |
| 607 | Previously unsubscribed | Address on unsubscribe list |
| 650 | Account disabled | Contact Mailgun support |
</mailgun_internal>

<smtp_codes>
**SMTP Response Codes:**

| Code | Category | Meaning |
|------|----------|---------|
| 250 | Success | Message accepted |
| 421 | Temporary | Service unavailable, try later |
| 450 | Temporary | Mailbox unavailable (try later) |
| 451 | Temporary | Server error, try later |
| 452 | Temporary | Insufficient storage |
| 500 | Permanent | Syntax error |
| 501 | Permanent | Syntax error in parameters |
| 502 | Permanent | Command not implemented |
| 503 | Permanent | Bad sequence of commands |
| 550 | Permanent | Mailbox unavailable (doesn't exist) |
| 551 | Permanent | User not local |
| 552 | Permanent | Storage exceeded |
| 553 | Permanent | Mailbox name not allowed |
| 554 | Permanent | Transaction failed |
</smtp_codes>
</delivery_status_codes>

<enhanced_status_codes>
## Enhanced Status Codes (5.x.x)

<address_status>
**Address Status (5.1.x):**

| Code | Meaning | Action |
|------|---------|--------|
| 5.1.0 | Other address status | Check address format |
| 5.1.1 | Bad destination mailbox | Address doesn't exist, remove |
| 5.1.2 | Bad destination system | Domain doesn't exist |
| 5.1.3 | Bad destination syntax | Invalid address format |
| 5.1.4 | Ambiguous address | Multiple matches |
| 5.1.5 | Valid but no mailbox | Catch-all rejected |
| 5.1.6 | Moved, no forwarding | Address relocated |
| 5.1.7 | Bad sender's mailbox | Sender address issue |
| 5.1.8 | Bad sender's system | Sender domain issue |
</address_status>

<mailbox_status>
**Mailbox Status (5.2.x):**

| Code | Meaning | Action |
|------|---------|--------|
| 5.2.0 | Other mailbox status | General mailbox issue |
| 5.2.1 | Mailbox disabled | Account suspended |
| 5.2.2 | Mailbox full | Quota exceeded |
| 5.2.3 | Message too long | Reduce message size |
| 5.2.4 | Mailing list expansion | List issue |
</mailbox_status>

<security_status>
**Security/Policy Status (5.7.x):**

| Code | Meaning | Action |
|------|---------|--------|
| 5.7.0 | Other security status | General security issue |
| 5.7.1 | Delivery not authorized | Blocked as spam/policy |
| 5.7.2 | Mailing list expansion prohibited | List blocked |
| 5.7.3 | Security conversion required | Encryption needed |
| 5.7.4 | Security features not supported | Incompatible |
| 5.7.5 | Cryptographic failure | TLS/encryption issue |
| 5.7.6 | Cryptographic algorithm not supported | Update encryption |
| 5.7.7 | Message integrity failure | Message corrupted |
| 5.7.8 | Trust relationship required | Authentication needed |
| 5.7.9 | Authentication required | Need to authenticate |
</security_status>
</enhanced_status_codes>

<common_bounce_messages>
## Common Bounce Messages

<hard_bounces>
**Hard Bounces (Permanent):**

| Message | Meaning | Action |
|---------|---------|--------|
| "User unknown" | Address doesn't exist | Remove from list |
| "Mailbox not found" | Account deleted | Remove from list |
| "No such user" | Invalid address | Remove from list |
| "Invalid recipient" | Address rejected | Remove from list |
| "Account disabled" | User suspended | Remove from list |
| "Domain not found" | Invalid domain | Remove from list |
</hard_bounces>

<soft_bounces>
**Soft Bounces (Temporary):**

| Message | Meaning | Action |
|---------|---------|--------|
| "Mailbox full" | Quota exceeded | Retry later |
| "Try again later" | Server busy | Auto-retry |
| "Connection timed out" | Network issue | Auto-retry |
| "Rate limit exceeded" | Sending too fast | Slow down |
| "Greylisting" | First-time delay | Auto-retry |
</soft_bounces>

<spam_blocks>
**Spam/Reputation Blocks:**

| Message | Meaning | Action |
|---------|---------|--------|
| "Blocked for spam" | Content flagged | Review content |
| "IP blacklisted" | IP reputation | Check blacklists |
| "Sender rejected" | Domain reputation | Warm up domain |
| "Message refused" | Policy violation | Review message |
| "Access denied" | Blocked sender | Check reputation |
</spam_blocks>
</common_bounce_messages>

<handling_errors>
## Error Handling Patterns

<node_pattern>
**Node.js:**
```javascript
try {
  const result = await mg.messages.create(domain, data);
  return { success: true, id: result.id };
} catch (error) {
  const statusCode = error.status || 500;
  const message = error.message || 'Unknown error';

  // Log for debugging
  console.error(`Mailgun error [${statusCode}]: ${message}`);

  // Handle specific errors
  switch (statusCode) {
    case 400:
      return { success: false, error: 'Invalid request', details: message };
    case 401:
      return { success: false, error: 'Authentication failed' };
    case 402:
      return { success: false, error: 'Account disabled' };
    case 429:
      // Implement retry with backoff
      return { success: false, error: 'Rate limited', retry: true };
    default:
      return { success: false, error: message };
  }
}
```
</node_pattern>

<laravel_pattern>
**Laravel:**
```php
use Symfony\Component\Mailer\Exception\TransportExceptionInterface;

try {
    Mail::to($user)->send(new WelcomeMail());
    return ['success' => true];
} catch (TransportExceptionInterface $e) {
    Log::error('Mail error', [
        'message' => $e->getMessage(),
        'recipient' => $user->email
    ]);

    return ['success' => false, 'error' => $e->getMessage()];
}
```
</laravel_pattern>
</handling_errors>
