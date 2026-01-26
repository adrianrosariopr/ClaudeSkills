---
name: stripe-skill
description: Comprehensive Stripe payment expertise from integration through optimization. Covers setup, subscriptions, one-time payments, webhooks, billing portal, testing, go-live, plus best practices, anti-patterns, pricing strategies, use cases, optimization techniques, and tips. Supports Laravel, Next.js, Nuxt, and SvelteKit. Use for building payment functionality, troubleshooting, or learning Stripe patterns.
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

**Implementation:**
1. **Set up Stripe integration** - Install packages, configure environment
2. **Implement subscriptions** - Recurring billing with plans
3. **Implement one-time payments** - Single product purchases
4. **Set up webhooks** - Handle Stripe events
5. **Add billing portal** - Customer self-service
6. **Test the integration** - Test cards, Stripe CLI
7. **Go live** - Production checklist

**Troubleshooting:**
8. **Debug an issue** - Troubleshoot payment problems

**Knowledge & Strategy:**
9. **Best practices** - Recommended patterns and approaches
10. **Use cases** - Common implementation scenarios (SaaS, marketplace, metered, etc.)
11. **Pricing strategies** - Subscription models, metered billing, regional pricing
12. **Optimization** - Reduce fees, improve conversion, reduce churn
13. **Tips and tricks** - CLI shortcuts, Cashier tips, dashboard tricks
14. **Anti-patterns** - What NOT to do (learn from common mistakes)

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

**Implementation workflows:**

| Response | Workflow |
|----------|----------|
| 1, "setup", "install", "configure" | `workflows/setup-integration.md` |
| 2, "subscription", "recurring", "plans" | `workflows/implement-subscriptions.md` |
| 3, "one-time", "single", "product" | `workflows/implement-one-time-payments.md` |
| 4, "webhook", "events" | `workflows/implement-webhooks.md` |
| 5, "portal", "billing", "manage" | `workflows/implement-billing-portal.md` |
| 6, "test", "testing", "stripe cli" | `workflows/test-integration.md` |
| 7, "live", "production", "deploy" | `workflows/go-live.md` |
| 8, "debug", "fix", "error", "issue" | `workflows/debug-stripe.md` |

**Knowledge references (read directly, no workflow):**

| Response | Reference |
|----------|-----------|
| 9, "best practice", "recommended", "good patterns" | `references/core/best-practices.md` |
| 10, "use case", "scenario", "example", "how to implement" | `references/core/use-cases.md` |
| 11, "pricing", "strategy", "metered", "per-seat", "freemium" | `references/core/pricing-strategies.md` |
| 12, "optimize", "optimization", "conversion", "churn", "fees" | `references/core/optimization.md` |
| 13, "tips", "tricks", "shortcuts", "cli tips" | `references/core/tips-and-tricks.md` |
| 14, "anti-pattern", "bad", "avoid", "don't", "mistake" | `references/core/anti-patterns.md` |

**After identifying framework and task:**
1. For implementation tasks: Read framework-specific references, then workflow file
2. For knowledge tasks: Read the reference file directly
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

**Implementation:**
- stripe-checkout.md - Checkout sessions and options
- subscriptions.md - Subscription lifecycle
- webhooks.md - Webhook handling patterns
- customer-portal.md - Portal configuration
- one-time-payments.md - Single purchase patterns
- security.md - PCI, API keys, verification
- testing.md - Test cards, Stripe CLI
- troubleshooting.md - Common issues

**Strategy & Knowledge:**
- best-practices.md - Recommended patterns and approaches
- anti-patterns.md - What NOT to do
- use-cases.md - Common scenarios (SaaS, marketplace, metered, etc.)
- pricing-strategies.md - Subscription models, tiered, regional pricing
- optimization.md - Reduce fees, improve conversion, reduce churn
- tips-and-tricks.md - CLI shortcuts, Cashier tips, dashboard tricks

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
