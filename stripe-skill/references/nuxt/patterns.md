<overview>
Nuxt 3 Stripe integration using @stripe/stripe-js for frontend and Nitro server routes for backend. Vue 3 Composition API patterns.
</overview>

<packages>
```bash
npm install stripe @stripe/stripe-js
```

| Package | Purpose | Where |
|---------|---------|-------|
| `stripe` | Server-side Stripe SDK | server/api/ routes |
| `@stripe/stripe-js` | Client-side Stripe.js loader | Frontend |

**Note:** No official Vue/Nuxt Stripe component library. Use @stripe/stripe-js directly with Vue refs.
</packages>

<stripe_composable>
Create a composable for Stripe:

```typescript
// composables/useStripe.ts
import { loadStripe, Stripe } from '@stripe/stripe-js';

let stripePromise: Promise<Stripe | null>;

export function useStripe() {
  const config = useRuntimeConfig();

  if (!stripePromise) {
    stripePromise = loadStripe(config.public.stripeKey);
  }

  return {
    stripePromise,
    getStripe: () => stripePromise,
  };
}
```
</stripe_composable>

<checkout_redirect>
**Recommended: Redirect to Stripe Checkout**

**Server route:**
```typescript
// server/api/checkout.post.ts
import Stripe from 'stripe';

export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig();
  const stripe = new Stripe(config.stripeSecretKey);
  const { priceId } = await readBody(event);

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${config.public.appUrl}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${config.public.appUrl}/pricing`,
  });

  return { url: session.url };
});
```

**Client component:**
```vue
<!-- pages/pricing.vue -->
<script setup lang="ts">
async function handleCheckout(priceId: string) {
  const { data } = await useFetch('/api/checkout', {
    method: 'POST',
    body: { priceId },
  });

  if (data.value?.url) {
    navigateTo(data.value.url, { external: true });
  }
}
</script>

<template>
  <button @click="handleCheckout('price_xxx')">
    Subscribe
  </button>
</template>
```
</checkout_redirect>

<payment_element>
**Embedded payment form with Vue:**

```vue
<!-- components/CheckoutForm.vue -->
<script setup lang="ts">
import { loadStripe, Stripe, StripeElements } from '@stripe/stripe-js';

const config = useRuntimeConfig();

const stripe = ref<Stripe | null>(null);
const elements = ref<StripeElements | null>(null);
const paymentElement = ref<HTMLDivElement | null>(null);
const error = ref<string | null>(null);
const processing = ref(false);

onMounted(async () => {
  // Load Stripe
  stripe.value = await loadStripe(config.public.stripeKey);

  // Get client secret from server
  const { data } = await useFetch('/api/create-payment-intent', {
    method: 'POST',
    body: { amount: 2000 },
  });

  if (!stripe.value || !data.value?.clientSecret) return;

  // Create Elements
  elements.value = stripe.value.elements({
    clientSecret: data.value.clientSecret,
    appearance: { theme: 'stripe' },
  });

  // Mount PaymentElement
  const payment = elements.value.create('payment');
  payment.mount(paymentElement.value!);
});

async function handleSubmit() {
  if (!stripe.value || !elements.value) return;

  processing.value = true;
  error.value = null;

  const { error: submitError } = await elements.value.submit();
  if (submitError) {
    error.value = submitError.message ?? 'Validation error';
    processing.value = false;
    return;
  }

  const { error: confirmError } = await stripe.value.confirmPayment({
    elements: elements.value,
    confirmParams: {
      return_url: `${window.location.origin}/success`,
    },
  });

  if (confirmError) {
    error.value = confirmError.message ?? 'Payment failed';
    processing.value = false;
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <div ref="paymentElement"></div>

    <p v-if="error" class="text-red-500">{{ error }}</p>

    <button :disabled="processing || !stripe">
      {{ processing ? 'Processing...' : 'Pay' }}
    </button>
  </form>
</template>
```

**Server route:**
```typescript
// server/api/create-payment-intent.post.ts
import Stripe from 'stripe';

export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig();
  const stripe = new Stripe(config.stripeSecretKey);
  const { amount } = await readBody(event);

  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency: 'usd',
    automatic_payment_methods: { enabled: true },
  });

  return { clientSecret: paymentIntent.client_secret };
});
```
</payment_element>

<card_element>
**Using CardElement:**

```vue
<script setup lang="ts">
import { loadStripe, Stripe, StripeCardElement } from '@stripe/stripe-js';

const config = useRuntimeConfig();

const stripe = ref<Stripe | null>(null);
const cardElement = ref<StripeCardElement | null>(null);
const cardContainer = ref<HTMLDivElement | null>(null);

onMounted(async () => {
  stripe.value = await loadStripe(config.public.stripeKey);

  if (!stripe.value) return;

  const elements = stripe.value.elements();
  cardElement.value = elements.create('card', {
    style: {
      base: {
        fontSize: '16px',
        color: '#32325d',
      },
    },
  });
  cardElement.value.mount(cardContainer.value!);
});

async function handleSubmit() {
  if (!stripe.value || !cardElement.value) return;

  const { error, paymentMethod } = await stripe.value.createPaymentMethod({
    type: 'card',
    card: cardElement.value,
  });

  if (error) {
    console.error(error);
  } else {
    // Send to server
    await $fetch('/api/process-payment', {
      method: 'POST',
      body: { paymentMethodId: paymentMethod.id },
    });
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <div ref="cardContainer"></div>
    <button>Pay</button>
  </form>
</template>
```
</card_element>

<webhooks>
```typescript
// server/api/webhooks/stripe.post.ts
import Stripe from 'stripe';

export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig();
  const stripe = new Stripe(config.stripeSecretKey);

  const body = await readRawBody(event);
  const signature = getHeader(event, 'stripe-signature');

  let stripeEvent: Stripe.Event;

  try {
    stripeEvent = stripe.webhooks.constructEvent(
      body!,
      signature!,
      config.stripeWebhookSecret
    );
  } catch (err) {
    console.error('Webhook signature verification failed');
    throw createError({ statusCode: 400, message: 'Invalid signature' });
  }

  switch (stripeEvent.type) {
    case 'checkout.session.completed':
      const session = stripeEvent.data.object;
      // Handle checkout completion
      break;

    case 'customer.subscription.updated':
      const subscription = stripeEvent.data.object;
      // Handle subscription update
      break;

    case 'invoice.payment_failed':
      const invoice = stripeEvent.data.object;
      // Handle failed payment
      break;
  }

  return { received: true };
});
```
</webhooks>

<customer_portal>
```typescript
// server/api/portal.post.ts
import Stripe from 'stripe';

export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig();
  const stripe = new Stripe(config.stripeSecretKey);
  const { customerId } = await readBody(event);

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${config.public.appUrl}/billing`,
  });

  return { url: session.url };
});
```
</customer_portal>

<runtime_config>
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

```env
# .env
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
NUXT_PUBLIC_STRIPE_KEY=pk_test_xxx
NUXT_PUBLIC_APP_URL=http://localhost:3000
```
</runtime_config>

<vue_stripe_plugin>
**Optional: Create a Nuxt plugin:**

```typescript
// plugins/stripe.client.ts
import { loadStripe } from '@stripe/stripe-js';

export default defineNuxtPlugin(async () => {
  const config = useRuntimeConfig();
  const stripe = await loadStripe(config.public.stripeKey);

  return {
    provide: {
      stripe,
    },
  };
});
```

**Usage:**
```vue
<script setup>
const { $stripe } = useNuxtApp();
</script>
```
</vue_stripe_plugin>

<context7_queries>
For latest Nuxt/Vue patterns:
```
mcp__context7__query-docs:
  libraryId: "/websites/stripe"
  query: "Vue Nuxt integration"
```
</context7_queries>
