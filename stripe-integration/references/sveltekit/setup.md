<overview>
SvelteKit Stripe setup using svelte-stripe components and Nitro server routes.
</overview>

<installation>
```bash
npm install stripe @stripe/stripe-js svelte-stripe
```
</installation>

<environment>
```env
# .env
PUBLIC_STRIPE_KEY=pk_test_xxx
SECRET_STRIPE_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
PUBLIC_URL=http://localhost:5173
```

**Access:**
```typescript
// Server (private)
import { SECRET_STRIPE_KEY } from '$env/static/private';

// Client (public)
import { PUBLIC_STRIPE_KEY } from '$env/static/public';
```
</environment>

<stripe_server>
```typescript
// src/lib/server/stripe.ts
import Stripe from 'stripe';
import { SECRET_STRIPE_KEY } from '$env/static/private';

export const stripe = new Stripe(SECRET_STRIPE_KEY, {
  apiVersion: '2024-06-20',
});
```
</stripe_server>

<directory_structure>
```
src/
├── lib/
│   └── server/
│       └── stripe.ts           # Server-side Stripe instance
├── routes/
│   ├── api/
│   │   ├── checkout/
│   │   │   └── +server.ts      # Checkout session
│   │   ├── create-payment-intent/
│   │   │   └── +server.ts      # PaymentIntent
│   │   ├── portal/
│   │   │   └── +server.ts      # Customer portal
│   │   └── webhooks/
│   │       └── stripe/
│   │           └── +server.ts  # Webhook handler
│   ├── checkout/
│   │   └── +page.svelte        # Checkout page
│   ├── success/
│   │   └── +page.svelte        # Success page
│   └── billing/
│       └── +page.svelte        # Billing page
```
</directory_structure>

<webhook_local_testing>
```bash
stripe listen --forward-to localhost:5173/api/webhooks/stripe
```
</webhook_local_testing>

<svelte_stripe_components>
Available from `svelte-stripe`:
- `Elements` - Provider
- `PaymentElement` - All-in-one payment
- `CardElement` - Card only
- `CardNumber`, `CardExpiry`, `CardCvc` - Split inputs
- `LinkAuthenticationElement` - Link auth
- `AddressElement` - Address collection
- `PaymentRequestButton` - Apple/Google Pay
</svelte_stripe_components>
