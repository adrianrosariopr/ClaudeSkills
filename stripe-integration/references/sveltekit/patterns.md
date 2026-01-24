<overview>
SvelteKit Stripe integration using svelte-stripe for frontend components and server endpoints for backend logic.
</overview>

<packages>
```bash
npm install stripe @stripe/stripe-js svelte-stripe
```

| Package | Purpose | Where |
|---------|---------|-------|
| `stripe` | Server-side Stripe SDK | +server.ts endpoints |
| `@stripe/stripe-js` | Client-side Stripe.js loader | Frontend |
| `svelte-stripe` | Svelte components | Frontend |
</packages>

<stripe_initialization>
```svelte
<!-- src/routes/checkout/+page.svelte -->
<script>
  import { loadStripe } from '@stripe/stripe-js';
  import { Elements } from 'svelte-stripe';
  import { onMount } from 'svelte';
  import { PUBLIC_STRIPE_KEY } from '$env/static/public';

  let stripe = null;

  onMount(async () => {
    stripe = await loadStripe(PUBLIC_STRIPE_KEY);
  });
</script>

{#if stripe}
  <Elements {stripe}>
    <!-- Payment components go here -->
  </Elements>
{:else}
  <p>Loading...</p>
{/if}
```
</stripe_initialization>

<checkout_redirect>
**Recommended: Redirect to Stripe Checkout**

**Server endpoint:**
```typescript
// src/routes/api/checkout/+server.ts
import { json } from '@sveltejs/kit';
import Stripe from 'stripe';
import { SECRET_STRIPE_KEY } from '$env/static/private';

const stripe = new Stripe(SECRET_STRIPE_KEY);

export async function POST({ request }) {
  const { priceId } = await request.json();

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.PUBLIC_URL}/pricing`,
  });

  return json({ url: session.url });
}
```

**Client:**
```svelte
<script>
  async function handleCheckout(priceId) {
    const response = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ priceId }),
    });
    const { url } = await response.json();
    window.location.href = url;
  }
</script>

<button on:click={() => handleCheckout('price_xxx')}>
  Subscribe
</button>
```
</checkout_redirect>

<payment_element>
**Embedded payment form:**

```svelte
<!-- src/routes/checkout/+page.svelte -->
<script>
  import { loadStripe } from '@stripe/stripe-js';
  import { Elements, PaymentElement } from 'svelte-stripe';
  import { onMount } from 'svelte';
  import { PUBLIC_STRIPE_KEY } from '$env/static/public';

  let stripe = null;
  let elements;
  let clientSecret = '';
  let processing = false;
  let error = null;

  onMount(async () => {
    stripe = await loadStripe(PUBLIC_STRIPE_KEY);

    // Get client secret from server
    const response = await fetch('/api/create-payment-intent', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount: 2000 }),
    });
    const data = await response.json();
    clientSecret = data.clientSecret;
  });

  async function submit() {
    if (!stripe || !elements) return;

    processing = true;
    error = null;

    const result = await stripe.confirmPayment({
      elements,
      redirect: 'if_required',
    });

    if (result.error) {
      error = result.error.message;
      processing = false;
    } else {
      window.location.href = '/success';
    }
  }
</script>

{#if stripe && clientSecret}
  <form on:submit|preventDefault={submit}>
    <Elements {stripe} {clientSecret} bind:elements>
      <PaymentElement />
    </Elements>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    <button disabled={processing}>
      {processing ? 'Processing...' : 'Pay'}
    </button>
  </form>
{:else}
  <p>Loading payment form...</p>
{/if}
```

**Server endpoint:**
```typescript
// src/routes/api/create-payment-intent/+server.ts
import { json } from '@sveltejs/kit';
import Stripe from 'stripe';
import { SECRET_STRIPE_KEY } from '$env/static/private';

const stripe = new Stripe(SECRET_STRIPE_KEY);

export async function POST({ request }) {
  const { amount } = await request.json();

  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency: 'usd',
    automatic_payment_methods: { enabled: true },
  });

  return json({ clientSecret: paymentIntent.client_secret });
}
```
</payment_element>

<card_element>
**Using CardElement:**

```svelte
<script>
  import { loadStripe } from '@stripe/stripe-js';
  import { Elements, CardElement } from 'svelte-stripe';
  import { onMount } from 'svelte';
  import { PUBLIC_STRIPE_KEY } from '$env/static/public';

  let stripe = null;
  let cardElement;

  onMount(async () => {
    stripe = await loadStripe(PUBLIC_STRIPE_KEY);
  });

  async function submit() {
    const { error, paymentMethod } = await stripe.createPaymentMethod({
      type: 'card',
      card: cardElement,
    });

    if (error) {
      console.error(error);
    } else {
      // Send paymentMethod.id to server
      await fetch('/api/process-payment', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ paymentMethodId: paymentMethod.id }),
      });
    }
  }
</script>

{#if stripe}
  <form on:submit|preventDefault={submit}>
    <Elements {stripe}>
      <CardElement bind:element={cardElement} />
    </Elements>
    <button>Pay</button>
  </form>
{/if}
```
</card_element>

<webhooks>
```typescript
// src/routes/api/webhooks/stripe/+server.ts
import { json } from '@sveltejs/kit';
import Stripe from 'stripe';
import { SECRET_STRIPE_KEY, STRIPE_WEBHOOK_SECRET } from '$env/static/private';

const stripe = new Stripe(SECRET_STRIPE_KEY);

export async function POST({ request }) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature');

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(body, signature!, STRIPE_WEBHOOK_SECRET);
  } catch (err) {
    console.error('Webhook signature verification failed');
    return json({ error: 'Invalid signature' }, { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object;
      // Handle checkout completion
      break;

    case 'customer.subscription.updated':
      const subscription = event.data.object;
      // Handle subscription update
      break;

    case 'invoice.payment_failed':
      const invoice = event.data.object;
      // Handle failed payment
      break;
  }

  return json({ received: true });
}
```
</webhooks>

<customer_portal>
```typescript
// src/routes/api/portal/+server.ts
import { json } from '@sveltejs/kit';
import Stripe from 'stripe';
import { SECRET_STRIPE_KEY } from '$env/static/private';

const stripe = new Stripe(SECRET_STRIPE_KEY);

export async function POST({ request }) {
  const { customerId } = await request.json();

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${process.env.PUBLIC_URL}/billing`,
  });

  return json({ url: session.url });
}
```
</customer_portal>

<environment_variables>
```env
# .env
PUBLIC_STRIPE_KEY=pk_test_xxx
SECRET_STRIPE_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

**Access in code:**
```typescript
// Server (private)
import { SECRET_STRIPE_KEY } from '$env/static/private';

// Client (public)
import { PUBLIC_STRIPE_KEY } from '$env/static/public';
```
</environment_variables>

<available_components>
From `svelte-stripe`:

| Component | Purpose |
|-----------|---------|
| `Elements` | Provider component |
| `PaymentElement` | All-in-one payment UI |
| `CardElement` | Card input only |
| `CardNumber`, `CardExpiry`, `CardCvc` | Split card inputs |
| `LinkAuthenticationElement` | Link authentication |
| `AddressElement` | Address collection |
| `PaymentRequestButton` | Apple/Google Pay |
</available_components>

<context7_queries>
For latest SvelteKit Stripe patterns:
```
mcp__context7__query-docs:
  libraryId: "/websites/sveltestripe"
  query: "your specific question"

mcp__context7__query-docs:
  libraryId: "/sveltejs/kit"
  query: "API routes server endpoints"
```
</context7_queries>
