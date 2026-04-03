---
name: stripe-payments
description: Stripe payment integration best practices — Checkout, Payment Intents, webhooks, subscriptions, SDK upgrades. Auto-activates for Stripe payment or billing code.
origin: VoltAgent/awesome-agent-skills (stripe)
tools: Read, Write, Edit, Bash
---

# Stripe Payments Skills

Official skills from the Stripe team covering integration best practices and SDK upgrades.

## Setup

```bash
npm install stripe @stripe/stripe-js
```

```ts
// lib/stripe.ts — server-side singleton
import Stripe from 'stripe';
export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-12-18.acacia',
  typescript: true,
});
```

## Checkout Session (Hosted)

```ts
// Create checkout session
const session = await stripe.checkout.sessions.create({
  mode: 'payment',              // or 'subscription'
  payment_method_types: ['card'],
  line_items: [
    {
      price_data: {
        currency: 'usd',
        product_data: { name: 'T-Shirt' },
        unit_amount: 2000,       // $20.00 in cents
      },
      quantity: 1,
    },
  ],
  success_url: `${process.env.NEXT_PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
  cancel_url: `${process.env.NEXT_PUBLIC_URL}/cancel`,
  metadata: { userId: '123' },  // pass custom data
});

return redirect(session.url!);
```

## Payment Intents (Custom UI)

```ts
// Server: create payment intent
const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000,
  currency: 'usd',
  automatic_payment_methods: { enabled: true },
  metadata: { orderId: order.id },
});

return { clientSecret: paymentIntent.client_secret };
```

```tsx
// Client: Stripe Elements
import { loadStripe } from '@stripe/stripe-js';
import { Elements, PaymentElement, useStripe, useElements } from '@stripe/react-stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PK!);

function CheckoutForm() {
  const stripe = useStripe();
  const elements = useElements();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!stripe || !elements) return;

    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: { return_url: `${window.location.origin}/success` },
    });

    if (error) console.error(error.message);
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      <button type="submit" disabled={!stripe}>Pay</button>
    </form>
  );
}

export function StripeCheckout({ clientSecret }: { clientSecret: string }) {
  return (
    <Elements stripe={stripePromise} options={{ clientSecret }}>
      <CheckoutForm />
    </Elements>
  );
}
```

## Subscriptions

```ts
// Create customer + subscription
const customer = await stripe.customers.create({
  email: user.email,
  metadata: { userId: user.id },
});

const subscription = await stripe.subscriptions.create({
  customer: customer.id,
  items: [{ price: process.env.STRIPE_PRICE_ID }],
  payment_behavior: 'default_incomplete',
  expand: ['latest_invoice.payment_intent'],
});

// Access client secret for confirmation
const invoice = subscription.latest_invoice as Stripe.Invoice;
const paymentIntent = invoice.payment_intent as Stripe.PaymentIntent;
return { clientSecret: paymentIntent.client_secret };
```

## Webhooks

```ts
// app/api/stripe/webhook/route.ts
import { stripe } from '@/lib/stripe';
import { headers } from 'next/headers';

export async function POST(request: Request) {
  const body = await request.text();
  const signature = headers().get('stripe-signature')!;

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(body, signature, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch (err) {
    return new Response('Invalid signature', { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed': {
      const session = event.data.object as Stripe.Checkout.Session;
      await fulfillOrder(session.metadata!.orderId);
      break;
    }
    case 'customer.subscription.updated': {
      const subscription = event.data.object as Stripe.Subscription;
      await updateSubscriptionStatus(subscription.id, subscription.status);
      break;
    }
    case 'invoice.payment_failed': {
      const invoice = event.data.object as Stripe.Invoice;
      await notifyPaymentFailed(invoice.customer as string);
      break;
    }
  }

  return new Response('OK');
}
```

```bash
# Local webhook testing
stripe listen --forward-to localhost:3000/api/stripe/webhook

# Trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
```

## Customer Portal

```ts
// Let customers manage their subscription
const session = await stripe.billingPortal.sessions.create({
  customer: customerId,
  return_url: `${process.env.NEXT_PUBLIC_URL}/dashboard`,
});

return redirect(session.url);
```

## Best Practices

1. **Always verify webhook signatures** — never trust raw webhook data
2. **Use idempotency keys** for retryable requests:
   ```ts
   await stripe.paymentIntents.create(params, {
     idempotencyKey: `order-${orderId}`,
   });
   ```
3. **Store Stripe IDs** in your DB (`stripe_customer_id`, `stripe_subscription_id`)
4. **Handle payment failures gracefully** — email customers, offer retry
5. **Use `expand`** to avoid extra API calls:
   ```ts
   await stripe.subscriptions.retrieve(id, { expand: ['latest_invoice.payment_intent'] });
   ```
6. **Test all webhook events** in staging before production
7. **Use Stripe Radar** rules for fraud prevention

## SDK Upgrade

```bash
# Upgrade Stripe Node SDK
npm install stripe@latest

# Check changelog for breaking changes
npx stripe-upgrade-helper
```

## Environment Variables

```env
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID=price_...
```

## Sources
- stripe/stripe-best-practices
- stripe/upgrade-stripe
