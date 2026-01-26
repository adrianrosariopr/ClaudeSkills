# Webapp Addendum Template

<usage>
Append this to the Core Spec for web applications. Describes user accounts, permissions, and key workflows.

**Audience:** Non-technical stakeholders. Write in plain language. No code, no jargon.
</usage>

<template>
```markdown

---

# How The App Works

## Logging In

**How users access their accounts:**
- {{Email and password, Google sign-in, magic email links, etc.}}

**How long they stay logged in:** {{e.g., Until they close the browser / For 7 days / Until they log out}}

### Login Steps

| Step | What Happens |
|------|--------------|
| 1 | User goes to the login page |
| 2 | {{What they see/do next}} |
| 3 | {{Continue...}} |

---

## User Types & Permissions

| User Type | What They Can Do |
|-----------|------------------|
| {{Admin}} | {{Plain description of admin capabilities}} |
| {{Regular User}} | {{Plain description of user capabilities}} |

---

## Main Screens

### {{Screen 1: e.g., Dashboard}}

{{What this screen is for and what the user sees when they land here}}

**What's on this screen:**
- {{Item}} - {{What it shows/does}}
- {{Item}} - {{What it shows/does}}

### {{Screen 2: e.g., Settings}}

{{Continue for key screens...}}

---

## How Users Do Things

### {{Task 1: e.g., Creating an Account}}

| Step | What Happens |
|------|--------------|
| 1 | {{What the user does/sees}} |
| 2 | {{What happens next}} |
| 3 | {{Continue...}} |

### {{Task 2: e.g., Making a Purchase}}

| Step | What Happens |
|------|--------------|
| 1 | {{Step}} |
| 2 | {{Step}} |

---

## Automatic Actions

Things that happen behind the scenes:

| What | When | Why |
|------|------|-----|
| {{e.g., Send welcome email}} | {{When someone signs up}} | {{Greets new users}} |
| {{e.g., Process payment}} | {{When subscription renews}} | {{Keeps access active}} |

---

## Connected Services

External services the app talks to:

| Service | What It Does For Us |
|---------|---------------------|
| {{Stripe}} | {{Handles payments and subscriptions}} |
| {{Mailgun}} | {{Sends emails to users}} |

---

*Webapp Details - Generated {{YYYY-MM-DD}}*
```
</template>

<guidelines>
1. **Zero jargon** - Replace "API endpoints" with "how the app gets data"
2. **User perspective** - "User clicks" not "System processes"
3. **Plain language** - "Automatic actions" not "Background jobs"
4. **Skip technical details** - No mention of REST, GraphQL, webhooks, etc.
5. **Focus on outcomes** - What users experience, not how it's implemented
</guidelines>
