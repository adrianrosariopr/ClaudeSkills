<overview>
Next.js Stripe setup for both App Router and Pages Router.
</overview>

<installation>
```bash
npm install stripe @stripe/stripe-js @stripe/react-stripe-js
```
</installation>

<environment>
```env
# .env.local
STRIPE_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_STRIPE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
NEXT_PUBLIC_URL=http://localhost:3000

# Price IDs
STRIPE_PRICE_BASIC=price_xxx
STRIPE_PRICE_PRO=price_xxx
```

**Note:** Only `NEXT_PUBLIC_*` variables are exposed to client.
</environment>

<stripe_client>
```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-06-20',
});
```

```typescript
// lib/stripe-client.ts
import { loadStripe } from '@stripe/stripe-js';

export const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_KEY!);
```
</stripe_client>

<app_router_provider>
```tsx
// app/providers.tsx
'use client';

import { Elements } from '@stripe/react-stripe-js';
import { stripePromise } from '@/lib/stripe-client';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <Elements stripe={stripePromise}>
      {children}
    </Elements>
  );
}
```

```tsx
// app/layout.tsx
import { Providers } from './providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```
</app_router_provider>

<pages_router_provider>
```tsx
// pages/_app.tsx
import { Elements } from '@stripe/react-stripe-js';
import { stripePromise } from '@/lib/stripe-client';

export default function App({ Component, pageProps }) {
  return (
    <Elements stripe={stripePromise}>
      <Component {...pageProps} />
    </Elements>
  );
}
```
</pages_router_provider>

<webhook_configuration>
**App Router:** Create `app/api/webhooks/stripe/route.ts`

**Pages Router:** Create `pages/api/webhooks/stripe.ts` with:
```typescript
export const config = {
  api: { bodyParser: false },
};
```

**Local testing:**
```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```
</webhook_configuration>

<directory_structure>
```
app/                          # or pages/
├── api/
│   ├── checkout/
│   │   └── route.ts          # Checkout session creation
│   ├── create-payment-intent/
│   │   └── route.ts          # PaymentIntent creation
│   ├── portal/
│   │   └── route.ts          # Customer portal
│   └── webhooks/
│       └── stripe/
│           └── route.ts      # Webhook handler
├── checkout/
│   └── page.tsx              # Checkout page
├── success/
│   └── page.tsx              # Success page
└── billing/
    └── page.tsx              # Billing management
lib/
├── stripe.ts                 # Server-side Stripe
└── stripe-client.ts          # Client-side Stripe
```
</directory_structure>

<typescript_types>
```typescript
// types/stripe.ts
export interface CheckoutRequest {
  priceId: string;
  quantity?: number;
}

export interface CheckoutResponse {
  url: string | null;
  error?: string;
}
```
</typescript_types>
