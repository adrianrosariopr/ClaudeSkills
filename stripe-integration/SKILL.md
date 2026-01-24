---
name: stripe-integration
description: Implement Stripe payment systems from scratch through production. Full lifecycle - setup, subscriptions, one-time payments, webhooks, billing portal, testing, and go-live. Supports Laravel, Next.js, Nuxt, and SvelteKit. Use when building payment functionality in any of these frameworks.
---

<essential_principles>

## How Stripe Integration Works

### 1. Stripe Checkout for Payment Collection

Always use Stripe Checkout (hosted payment page) for collecting payment details:
- PCI compliance (card data never touches your server)
- 3D Secure / SCA compliance built-in
- Mobile-optimized, supports multiple payment methods

### 2. Webhooks Are the Source of Truth

Never rely solely on redirect URLs for payment confirmation:
- User completes payment → redirected to success URL (unreliable)
- Stripe sends webhook → update your database (reliable)
- Always verify webhook signatures before processing

### 3. Environment Separation

Stripe maintains completely separate environments:
- Test mode: `pk_test_*`, `sk_test_*` keys
- Live mode: `pk_live_*`, `sk_live_*` keys
- Products/Prices are separate between modes
- Never mix test and live credentials

### 4. Framework-Specific Patterns

Each framework has its own patterns:
- **Laravel**: Use Laravel Cashier (full integration with Eloquent)
- **Next.js**: API routes + @stripe/react-stripe-js
- **Nuxt**: Server routes + @stripe/stripe-js with Vue
- **SvelteKit**: Server endpoints + svelte-stripe

</essential_principles>

<intake>

## What framework are you using?

1. **Laravel** (with Laravel Cashier)
2. **Next.js** (React)
3. **Nuxt** (Vue)
4. **SvelteKit** (Svelte)

**After framework selection, what would you like to do?**

1. **Set up Stripe integration** - Install packages, configure environment
2. **Implement subscriptions** - Recurring billing with plans
3. **Implement one-time payments** - Single product purchases
4. **Set up webhooks** - Handle Stripe events
5. **Add billing portal** - Customer self-service
6. **Debug an issue** - Troubleshoot payment problems
7. **Test the integration** - Test cards, Stripe CLI
8. **Go live** - Production checklist

**Wait for response before proceeding.**
</intake>

<routing>

## Framework Selection

| Response | Framework | References to Load |
|----------|-----------|-------------------|
| 1, "laravel", "php", "cashier" | Laravel | `references/laravel/setup.md` + `references/laravel/laravel-cashier.md` |
| 2, "next", "nextjs", "react" | Next.js | `references/nextjs/setup.md` + `references/nextjs/patterns.md` |
| 3, "nuxt", "vue" | Nuxt | `references/nuxt/setup.md` + `references/nuxt/patterns.md` |
| 4, "svelte", "sveltekit" | SvelteKit | `references/sveltekit/setup.md` + `references/sveltekit/patterns.md` |

## Task Routing

| Response | Workflow |
|----------|----------|
| 1, "setup", "install", "configure" | `workflows/setup-integration.md` |
| 2, "subscription", "recurring", "plans" | `workflows/implement-subscriptions.md` |
| 3, "one-time", "single", "product" | `workflows/implement-one-time-payments.md` |
| 4, "webhook", "events" | `workflows/implement-webhooks.md` |
| 5, "portal", "billing", "manage" | `workflows/implement-billing-portal.md` |
| 6, "debug", "fix", "error", "issue" | `workflows/debug-stripe.md` |
| 7, "test", "testing", "stripe cli" | `workflows/test-integration.md` |
| 8, "live", "production", "deploy" | `workflows/go-live.md` |

**After identifying framework and task:**
1. Read the framework-specific references first
2. Read the workflow file
3. Adapt workflow patterns to the selected framework (workflows have Laravel examples by default)

</routing>

<verification_loop>

## After Every Change

**All frameworks:**
```bash
# Forward webhooks locally
stripe listen --forward-to localhost:PORT/WEBHOOK_PATH

# Trigger test events
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
```

**Laravel:**
```bash
php artisan config:clear && php artisan config:cache
php artisan route:list | grep -E "(checkout|billing|stripe)"
```

**Next.js / Nuxt / SvelteKit:**
```bash
# Verify routes respond
curl -X POST http://localhost:PORT/api/checkout -d '{}'
```

</verification_loop>

<reference_index>

## Domain Knowledge

### Core (Framework-Agnostic)
All in `references/core/`:
- stripe-checkout.md - Checkout sessions and options
- subscriptions.md - Subscription lifecycle
- webhooks.md - Webhook handling patterns
- customer-portal.md - Portal configuration
- one-time-payments.md - Single purchase patterns
- security.md - PCI, API keys, verification
- testing.md - Test cards, Stripe CLI
- troubleshooting.md - Common issues
- anti-patterns.md - What NOT to do

### Laravel
All in `references/laravel/`:
- setup.md - Installation and configuration
- laravel-cashier.md - Cashier methods and patterns

### Next.js
All in `references/nextjs/`:
- setup.md - Installation and configuration
- patterns.md - API routes, hooks, components

### Nuxt
All in `references/nuxt/`:
- setup.md - Installation and configuration
- patterns.md - Server routes, composables

### SvelteKit
All in `references/sveltekit/`:
- setup.md - Installation and configuration
- patterns.md - Server endpoints, components

</reference_index>

<workflows_index>

## Workflows

All in `workflows/`:

| File | Purpose |
|------|---------|
| setup-integration.md | Install packages, configure env |
| implement-subscriptions.md | Add subscription plans and checkout |
| implement-one-time-payments.md | Add single product purchases |
| implement-webhooks.md | Handle Stripe webhook events |
| implement-billing-portal.md | Customer billing self-service |
| debug-stripe.md | Troubleshoot payment issues |
| test-integration.md | Test with Stripe CLI and test cards |
| go-live.md | Production deployment checklist |

**Note:** Workflows contain Laravel examples. Adapt using framework-specific references.

</workflows_index>

<context7_usage>

## Getting Current Documentation

**Laravel Cashier:**
```
mcp__context7__query-docs:
  libraryId: "/laravel/cashier-stripe"
  query: "your question"
```

**React/Next.js:**
```
mcp__context7__query-docs:
  libraryId: "/stripe/react-stripe-js"
  query: "your question"
```

**SvelteKit:**
```
mcp__context7__query-docs:
  libraryId: "/websites/sveltestripe"
  query: "your question"
```

**General Stripe API:**
```
mcp__context7__query-docs:
  libraryId: "/websites/stripe"
  query: "your question"
```

</context7_usage>

<quick_reference>

## Package Installation

| Framework | Command |
|-----------|---------|
| Laravel | `composer require laravel/cashier` |
| Next.js | `npm install stripe @stripe/stripe-js @stripe/react-stripe-js` |
| Nuxt | `npm install stripe @stripe/stripe-js` |
| SvelteKit | `npm install stripe @stripe/stripe-js svelte-stripe` |

## Environment Variables

| Variable | Laravel | Next.js | Nuxt | SvelteKit |
|----------|---------|---------|------|-----------|
| Public key | `STRIPE_KEY` | `NEXT_PUBLIC_STRIPE_KEY` | `NUXT_PUBLIC_STRIPE_KEY` | `PUBLIC_STRIPE_KEY` |
| Secret key | `STRIPE_SECRET` | `STRIPE_SECRET_KEY` | `STRIPE_SECRET_KEY` | `SECRET_STRIPE_KEY` |
| Webhook | `STRIPE_WEBHOOK_SECRET` | `STRIPE_WEBHOOK_SECRET` | `STRIPE_WEBHOOK_SECRET` | `STRIPE_WEBHOOK_SECRET` |

## Webhook Endpoints

| Framework | Path |
|-----------|------|
| Laravel | `/stripe/webhook` (auto-registered) |
| Next.js | `/api/webhooks/stripe` |
| Nuxt | `/api/webhooks/stripe` |
| SvelteKit | `/api/webhooks/stripe` |

</quick_reference>
