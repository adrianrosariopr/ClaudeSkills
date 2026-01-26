<overview>
Nuxt 3 Stripe setup using @stripe/stripe-js and Nitro server routes.
</overview>

<installation>
```bash
npm install stripe @stripe/stripe-js
```

**Note:** No official Vue/Nuxt Stripe component library. Use @stripe/stripe-js directly.
</installation>

<environment>
```env
# .env
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
NUXT_PUBLIC_STRIPE_KEY=pk_test_xxx
NUXT_PUBLIC_APP_URL=http://localhost:3000
```
</environment>

<nuxt_config>
```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    stripeSecretKey: process.env.STRIPE_SECRET_KEY,
    stripeWebhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
    public: {
      stripeKey: process.env.NUXT_PUBLIC_STRIPE_KEY,
      appUrl: process.env.NUXT_PUBLIC_APP_URL,
    },
  },
});
```
</nuxt_config>

<stripe_server>
```typescript
// server/utils/stripe.ts
import Stripe from 'stripe';

let stripe: Stripe;

export function useStripeServer() {
  if (!stripe) {
    const config = useRuntimeConfig();
    stripe = new Stripe(config.stripeSecretKey, {
      apiVersion: '2024-06-20',
    });
  }
  return stripe;
}
```
</stripe_server>

<stripe_client_plugin>
```typescript
// plugins/stripe.client.ts
import { loadStripe } from '@stripe/stripe-js';

export default defineNuxtPlugin(async () => {
  const config = useRuntimeConfig();
  const stripe = await loadStripe(config.public.stripeKey);

  return {
    provide: { stripe },
  };
});
```

**Usage:**
```vue
<script setup>
const { $stripe } = useNuxtApp();
</script>
```
</stripe_client_plugin>

<composable>
```typescript
// composables/useStripe.ts
import { loadStripe, Stripe } from '@stripe/stripe-js';

let stripePromise: Promise<Stripe | null>;

export function useStripe() {
  const config = useRuntimeConfig();

  if (!stripePromise) {
    stripePromise = loadStripe(config.public.stripeKey);
  }

  return { stripePromise };
}
```
</composable>

<directory_structure>
```
server/
├── api/
│   ├── checkout.post.ts         # Checkout session
│   ├── create-payment-intent.post.ts
│   ├── portal.post.ts           # Customer portal
│   └── webhooks/
│       └── stripe.post.ts       # Webhook handler
└── utils/
    └── stripe.ts                # Stripe instance

composables/
└── useStripe.ts                 # Client-side Stripe

plugins/
└── stripe.client.ts             # Optional plugin

pages/
├── checkout.vue
├── success.vue
└── billing.vue
```
</directory_structure>

<webhook_local_testing>
```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```
</webhook_local_testing>
