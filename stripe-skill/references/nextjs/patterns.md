<overview>
Next.js Stripe integration patterns using @stripe/react-stripe-js for frontend and API routes for backend. Supports both Pages Router and App Router.
</overview>

<packages>
```bash
npm install stripe @stripe/stripe-js @stripe/react-stripe-js
```

| Package | Purpose | Where |
|---------|---------|-------|
| `stripe` | Server-side Stripe SDK | API routes, server actions |
| `@stripe/stripe-js` | Client-side Stripe.js loader | Frontend |
| `@stripe/react-stripe-js` | React components & hooks | Frontend |
</packages>

<stripe_provider>
**App Router (layout.tsx):**
```tsx
// app/providers.tsx
'use client';

import { loadStripe } from '@stripe/stripe-js';
import { Elements } from '@stripe/react-stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_KEY!);

export function StripeProvider({ children }: { children: React.ReactNode }) {
  return (
    <Elements stripe={stripePromise}>
      {children}
    </Elements>
  );
}
```

**With options (for PaymentElement):**
```tsx
<Elements
  stripe={stripePromise}
  options={{
    mode: 'payment',
    amount: 1099,
    currency: 'usd',
    appearance: {
      theme: 'stripe',
    },
  }}
>
  {children}
</Elements>
```
</stripe_provider>

<checkout_redirect>
**Recommended approach - redirect to Stripe Checkout:**

```tsx
// app/api/checkout/route.ts (App Router)
import { NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(request: Request) {
  const { priceId } = await request.json();

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription', // or 'payment' for one-time
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_URL}/pricing`,
  });

  return NextResponse.json({ url: session.url });
}
```

**Client:**
```tsx
const handleCheckout = async (priceId: string) => {
  const res = await fetch('/api/checkout', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ priceId }),
  });
  const { url } = await res.json();
  window.location.href = url;
};
```
</checkout_redirect>

<embedded_checkout>
**Using Stripe Elements for embedded payment:**

```tsx
// components/CheckoutForm.tsx
'use client';

import { useStripe, useElements, PaymentElement } from '@stripe/react-stripe-js';
import { useState } from 'react';

export function CheckoutForm() {
  const stripe = useStripe();
  const elements = useElements();
  const [error, setError] = useState<string | null>(null);
  const [processing, setProcessing] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!stripe || !elements) return;

    setProcessing(true);

    // Validate form
    const { error: submitError } = await elements.submit();
    if (submitError) {
      setError(submitError.message ?? 'Validation error');
      setProcessing(false);
      return;
    }

    // Create PaymentIntent on server
    const res = await fetch('/api/create-payment-intent', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount: 2000 }),
    });
    const { clientSecret } = await res.json();

    // Confirm payment
    const { error } = await stripe.confirmPayment({
      elements,
      clientSecret,
      confirmParams: {
        return_url: `${window.location.origin}/success`,
      },
    });

    if (error) {
      setError(error.message ?? 'Payment failed');
      setProcessing(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      {error && <div className="text-red-500">{error}</div>}
      <button disabled={!stripe || processing}>
        {processing ? 'Processing...' : 'Pay'}
      </button>
    </form>
  );
}
```

**Server endpoint:**
```tsx
// app/api/create-payment-intent/route.ts
import { NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(request: Request) {
  const { amount } = await request.json();

  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency: 'usd',
    automatic_payment_methods: { enabled: true },
  });

  return NextResponse.json({ clientSecret: paymentIntent.client_secret });
}
```
</embedded_checkout>

<webhooks>
**App Router webhook handler:**

```tsx
// app/api/webhooks/stripe/route.ts
import { headers } from 'next/headers';
import { NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(request: Request) {
  const body = await request.text();
  const signature = headers().get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error('Webhook signature verification failed');
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object as Stripe.Checkout.Session;
      // Handle successful checkout
      await handleCheckoutComplete(session);
      break;

    case 'customer.subscription.created':
    case 'customer.subscription.updated':
      const subscription = event.data.object as Stripe.Subscription;
      await handleSubscriptionChange(subscription);
      break;

    case 'invoice.payment_failed':
      const invoice = event.data.object as Stripe.Invoice;
      await handlePaymentFailed(invoice);
      break;
  }

  return NextResponse.json({ received: true });
}
```

**Pages Router webhook handler:**
```tsx
// pages/api/webhooks/stripe.ts
import { buffer } from 'micro';
import type { NextApiRequest, NextApiResponse } from 'next';
import Stripe from 'stripe';

export const config = {
  api: { bodyParser: false }, // Required for signature verification
};

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }

  const buf = await buffer(req);
  const signature = req.headers['stripe-signature']!;

  try {
    const event = stripe.webhooks.constructEvent(
      buf,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );

    // Handle event...

    res.json({ received: true });
  } catch (err) {
    res.status(400).send(`Webhook Error: ${err}`);
  }
}
```
</webhooks>

<server_actions>
**App Router Server Actions (Next.js 14+):**

```tsx
// app/actions/stripe.ts
'use server';

import Stripe from 'stripe';
import { redirect } from 'next/navigation';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function createCheckoutSession(priceId: string) {
  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_URL}/success`,
    cancel_url: `${process.env.NEXT_PUBLIC_URL}/pricing`,
  });

  redirect(session.url!);
}

export async function createPortalSession(customerId: string) {
  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${process.env.NEXT_PUBLIC_URL}/billing`,
  });

  redirect(session.url);
}
```

**Usage:**
```tsx
<form action={createCheckoutSession.bind(null, priceId)}>
  <button type="submit">Subscribe</button>
</form>
```
</server_actions>

<environment_variables>
```env
# .env.local
STRIPE_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_STRIPE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# Price IDs
STRIPE_PRICE_BASIC=price_xxx
STRIPE_PRICE_PRO=price_xxx
```

**Note:** Only `NEXT_PUBLIC_*` variables are exposed to the client.
</environment_variables>

<customer_portal>
```tsx
// app/api/portal/route.ts
import { NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(request: Request) {
  const { customerId } = await request.json();

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${process.env.NEXT_PUBLIC_URL}/billing`,
  });

  return NextResponse.json({ url: session.url });
}
```
</customer_portal>

<context7_queries>
For latest Next.js Stripe patterns:
```
mcp__context7__query-docs:
  libraryId: "/stripe/react-stripe-js"
  query: "your specific question"

mcp__context7__query-docs:
  libraryId: "/websites/stripe"
  query: "Next.js integration webhooks API routes"
```
</context7_queries>
