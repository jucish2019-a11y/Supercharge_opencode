---
name: payments
description: Integrate Stripe for payments — one-time payments, subscriptions, webhooks, billing portals, invoices, and payment UI patterns
---

## What I do

I integrate payment processing using Stripe:

- **One-time payments** — Checkout sessions, payment intents, simple products
- **Subscriptions** — Recurring billing, plans, upgrades, downgrades, cancellations
- **Webhooks** — Reliable event processing, idempotency, retry handling
- **Billing portal** — Customer self-service for invoices, plans, payment methods
- **Pricing pages** — Plan comparison UI, free trials, coupons
- **Security** — Idempotency keys, webhook verification, PCI compliance via Stripe
- **Testing** — Stripe CLI, test cards, local webhook testing

## When to use me

Use this skill when:
- Adding payment processing to an application
- Implementing subscription billing
- Building a pricing page with plan selection
- Handling Stripe webhooks reliably
- Setting up customer billing portals
- Managing upgrades, downgrades, and cancellations

## Stripe integration overview

```
Client                  Server                  Stripe
  |                       |                       |
  | Create checkout       |                       |
  |--------------------->|                       |
  |                       | Create session       |
  |                       |--------------------->|
  |                       | Session URL          |
  |                       |<---------------------|
  | Redirect to Stripe    |                       |
  |<---------------------|                       |
  |                       |                       |
  | User completes payment on Stripe             |
  |                       |                       |
  |                       | Webhook: checkout.completed
  |                       |<---------------------|
  |                       | Fulfill order       |
  | Redirect to success   |                       |
  |<---------------------|                       |
```

## One-time payment

### Server: Create checkout session

```ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-06-20',
  typescript: true,
});

app.post('/api/checkout', async (req, res) => {
  const { priceId, quantity = 1 } = req.body;
  const userId = req.user.id;

  const session = await stripe.checkout.sessions.create({
    mode: 'payment',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity }],
    success_url: `${process.env.APP_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/pricing`,
    client_reference_id: userId,
    metadata: { userId },
  });

  res.json({ url: session.url });
});
```

### Client: Redirect to Stripe checkout

```tsx
function BuyButton({ priceId }: { priceId: string }) {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);
    try {
      const { url } = await fetch('/api/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ priceId }),
      }).then(r => r.json());
      window.location.href = url;
    } catch (error) {
      toast.error('Payment failed. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return <button onClick={handleCheckout} disabled={loading}>{loading ? 'Redirecting...' : 'Buy Now'}</button>;
}
```

## Subscriptions

### Pricing model

```ts
// Stripe Products and Prices (created in Stripe Dashboard or via API)

// Free plan
const FREE_PLAN = {
  name: 'Free',
  price: 0,
  features: ['5 projects', '1GB storage', 'Community support'],
  limits: { projects: 5, storage: '1GB' },
};

// Pro plan
const PRO_PLAN = {
  id: 'prod_pro',
  priceId: process.env.STRIPE_PRO_PRICE_ID!, // stripe price_xxx
  name: 'Pro',
  price: 29,
  interval: 'month',
  features: ['Unlimited projects', '100GB storage', 'Priority support', 'API access'],
  limits: { projects: -1, storage: '100GB' },
};

// Enterprise plan
const ENTERPRISE_PLAN = {
  id: 'prod_enterprise',
  priceId: process.env.STRIPE_ENTERPRISE_PRICE_ID!,
  name: 'Enterprise',
  price: 99,
  interval: 'month',
  features: ['Everything in Pro', 'Unlimited storage', 'Dedicated support', 'SSO', 'Audit logs'],
  limits: { projects: -1, storage: 'unlimited' },
};
```

### Create subscription

```ts
app.post('/api/subscribe', async (req, res) => {
  const { priceId } = req.body;
  const userId = req.user.id;
  const userEmail = req.user.email;

  const customer = await getOrCreateCustomer(userId, userEmail);

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    customer: customer.id,
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.APP_URL}/dashboard?upgraded=true`,
    cancel_url: `${process.env.APP_URL}/pricing`,
    subscription_data: {
      trial_period_days: 14, // Free trial
      metadata: { userId },
    },
  });

  res.json({ url: session.url });
});
```

### Customer model

```prisma
model User {
  id              String    @id @default(cuid())
  email           String    @unique
  name            String

  // Stripe fields
  stripeCustomerId    String?   @unique
  stripePriceId       String?   // Current plan price ID
  stripeSubscriptionId String?  @unique
  stripeCurrentPeriodEnd DateTime?

  subscriptionStatus  SubscriptionStatus @default(FREE)
}
```

```ts
async function getOrCreateCustomer(userId: string, email: string) {
  const user = await db.user.findUnique({ where: { id: userId } });

  if (user?.stripeCustomerId) {
    return { id: user.stripeCustomerId };
  }

  const customer = await stripe.customers.create({
    email,
    metadata: { userId },
  });

  await db.user.update({
    where: { id: userId },
    data: { stripeCustomerId: customer.id },
  });

  return customer;
}
```

### Upgrade / downgrade

```ts
// Upgrade or change plan
app.post('/api/subscription/change', async (req, res) => {
  const { newPriceId } = req.body;
  const user = req.user;

  if (!user.stripeSubscriptionId) {
    return res.status(400).json({ error: 'No active subscription' });
  }

  const subscription = await stripe.subscriptions.retrieve(user.stripeSubscriptionId);

  const updatedSubscription = await stripe.subscriptions.update(
    user.stripeSubscriptionId,
    {
      items: [{
        id: subscription.items.data[0].id,
        price: newPriceId,
      }],
      proration_behavior: 'create_prorations', // Prorate the difference
      metadata: { userId: user.id },
    }
  );

  // Update user plan in database (will be confirmed by webhook)
  await db.user.update({
    where: { id: user.id },
    data: { stripePriceId: newPriceId },
  });

  res.json({ subscription: updatedSubscription });
});

// Cancel subscription (at end of period)
app.post('/api/subscription/cancel', async (req, res) => {
  const user = req.user;

  if (!user.stripeSubscriptionId) {
    return res.status(400).json({ error: 'No active subscription' });
  }

  const subscription = await stripe.subscriptions.update(
    user.stripeSubscriptionId,
    { cancel_at_period_end: true }
  );

  res.json({ subscription });
});

// Resume cancelled subscription
app.post('/api/subscription/resume', async (req, res) => {
  const user = req.user;

  const subscription = await stripe.subscriptions.update(
    user.stripeSubscriptionId!,
    { cancel_at_period_end: false }
  );

  res.json({ subscription });
});
```

## Webhooks

### Why webhooks are critical

Webhooks are how Stripe tells your server about payment events. Without them, a user could pay on Stripe but your database wouldn't update.

**Critical rule**: Always use webhooks for order fulfillment, never rely on the client-side `success_url` redirect.

### Webhook handler

```ts
import Stripe from 'stripe';
import { buffer } from 'micro';

export const config = { api: { bodyParser: false } };

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export default async function handler(req: Request) {
  if (req.method !== 'POST') return new Response('Method not allowed', { status: 405 });

  const body = await req.text();
  const signature = req.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (err) {
    console.error('Webhook signature verification failed:', err);
    return new Response('Invalid signature', { status: 400 });
  }

  // Process the event
  try {
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutComplete(event.data.object as Stripe.Checkout.Session);
        break;

      case 'customer.subscription.created':
        await handleSubscriptionCreated(event.data.object as Stripe.Subscription);
        break;

      case 'customer.subscription.updated':
        await handleSubscriptionUpdated(event.data.object as Stripe.Subscription);
        break;

      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
        break;

      case 'invoice.payment_succeeded':
        await handlePaymentSucceeded(event.data.object as Stripe.Invoice);
        break;

      case 'invoice.payment_failed':
        await handlePaymentFailed(event.data.object as Stripe.Invoice);
        break;

      case 'customer.subscription.trial_will_end':
        await handleTrialEnding(event.data.object as Stripe.Subscription);
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }
  } catch (err) {
    console.error('Webhook handler error:', err);
    return new Response('Webhook handler error', { status: 500 });
  }

  return new Response(JSON.stringify({ received: true }), { status: 200 });
}
```

### Event handlers

```ts
async function handleCheckoutComplete(session: Stripe.Checkout.Session) {
  const userId = session.metadata?.userId;
  if (!userId) throw new Error('No userId in session metadata');

  // Fulfill the purchase
  if (session.mode === 'payment') {
    // One-time payment: grant access
    await db.user.update({
      where: { id: userId },
      data: {
        subscriptionStatus: 'ACTIVE',
        stripePriceId: 'price_onetime',
      },
    });
  }
  // Subscription: handled by subscription.created webhook
}

async function handleSubscriptionCreated(subscription: Stripe.Subscription) {
  const userId = subscription.metadata.userId;
  const priceId = subscription.items.data[0].price.id;

  await db.user.update({
    where: { id: userId },
    data: {
      subscriptionStatus: 'ACTIVE',
      stripePriceId: priceId,
      stripeSubscriptionId: subscription.id,
      stripeCurrentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });
}

async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  const userId = subscription.metadata.userId;
  const priceId = subscription.items.data[0].price.id;
  const status = subscription.status === 'active' ? 'ACTIVE' :
                subscription.status === 'trialing' ? 'TRIALING' :
                subscription.status === 'past_due' ? 'PAST_DUE' : 'INACTIVE';

  await db.user.update({
    where: { id: userId },
    data: {
      subscriptionStatus: status,
      stripePriceId: priceId,
      stripeCurrentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  const userId = subscription.metadata.userId;

  await db.user.update({
    where: { id: userId },
    data: {
      subscriptionStatus: 'INACTIVE',
      stripePriceId: null,
      stripeSubscriptionId: null,
      stripeCurrentPeriodEnd: null,
    },
  });
}

async function handlePaymentFailed(invoice: Stripe.Invoice) {
  const customerId = invoice.customer as string;
  const user = await db.user.findUnique({ where: { stripeCustomerId: customerId } });

  if (!user) return;

  // Notify user about failed payment
  await sendEmail(user.email, 'payment-failed', {
    attemptCount: invoice.attempt_count,
    nextAttempt: invoice.next_payment_attempt,
  });

  // After 3 failed attempts, mark subscription as past due
  if (invoice.attempt_count >= 3) {
    await db.user.update({
      where: { id: user.id },
      data: { subscriptionStatus: 'PAST_DUE' },
    });
  }
}
```

### Webhook idempotency

```ts
// Process each event exactly once
async function processEvent(event: Stripe.Event) {
  const eventId = event.id;

  // Check if already processed
  const existing = await db.webhookEvent.findUnique({ where: { id: eventId } });
  if (existing) {
    console.log(`Event ${eventId} already processed, skipping`);
    return;
  }

  // Process the event
  await handleEvent(event);

  // Mark as processed
  await db.webhookEvent.create({
    data: { id: eventId, type: event.type, processedAt: new Date() },
  });
}
```

## Billing portal

### Customer self-service

```ts
app.post('/api/billing/portal', async (req, res) => {
  const user = req.user;

  if (!user.stripeCustomerId) {
    return res.status(400).json({ error: 'No billing account' });
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: user.stripeCustomerId,
    return_url: `${process.env.APP_URL}/settings/billing`,
  });

  res.json({ url: session.url });
});
```

The Stripe Billing Portal provides:
- View invoices and payment history
- Update payment method
- Cancel subscription
- Change plan (if configured)

## Pricing page UI

```tsx
const plans = [
  {
    name: 'Free',
    price: 0,
    priceId: null,
    description: 'For individuals getting started',
    features: ['5 projects', '1GB storage', 'Community support'],
    cta: 'Current plan',
    highlighted: false,
  },
  {
    name: 'Pro',
    price: 29,
    priceId: process.env.NEXT_PUBLIC_STRIPE_PRO_PRICE_ID,
    description: 'For professionals and growing teams',
    features: ['Unlimited projects', '100GB storage', 'Priority support', 'API access'],
    cta: 'Start free trial',
    highlighted: true,
  },
  {
    name: 'Enterprise',
    price: 99,
    priceId: process.env.NEXT_PUBLIC_STRIPE_ENTERPRISE_PRICE_ID,
    description: 'For large teams with advanced needs',
    features: ['Everything in Pro', 'Unlimited storage', 'Dedicated support', 'SSO', 'Audit logs'],
    cta: 'Contact sales',
    highlighted: false,
  },
];

function PricingPage() {
  const { user } = useAuth();

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-5xl mx-auto px-6 py-16">
      {plans.map(plan => (
        <PricingCard
          key={plan.name}
          plan={plan}
          isCurrentPlan={user?.stripePriceId === plan.priceId}
        />
      ))}
    </div>
  );
}
```

## Testing

### Test cards

```
Successful payment:     4242 4242 4242 4242
Requires authentication: 4000 0025 0000 3155
Declined card:          4000 0000 0000 0002
Insufficient funds:     4000 0000 0000 9995
Lost card:              4000 0000 0000 9987
Processing error:        4000 0000 0000 0119

Any future expiry date, any CVC, any postal code.
```

### Local webhook testing

```bash
# Install Stripe CLI
stripe login

# Forward webhooks to local server
stripe listen --forward-to localhost:3000/api/stripe/webhook

# Trigger specific events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.created
stripe trigger customer.subscription.updated
stripe trigger customer.subscription.deleted
stripe trigger invoice.payment_failed
```

## Quality checklist

- [ ] Webhook signature verification on every event
- [ ] Idempotency key on every webhook event (deduplication)
- [ ] Customer created in Stripe before subscription
- [ ] Free trial via `trial_period_days` on subscription
- [ ] Proration on plan changes (upgrade credits, downgrade credits)
- [ ] Cancel at period end (not immediate cancellation)
- [ ] Subscription status persisted in database (via webhooks, not client)
- [ ] Failed payment handling (email notification, grace period, access revocation)
- [ ] Billing portal for customer self-service
- [ ] Test cards for manual testing
- [ ] Stripe CLI for local webhook testing
- [ ] Error handling: payment declined, network failure, webhook failure
- [ ] Rate limiting on checkout endpoints
- [ ] PCI compliance: never store card numbers (Stripe handles this)

## Anti-patterns I avoid

- Fulfilling orders on client-side success callback — use webhooks
- Storing credit card numbers — never, Stripe handles PCI
- Trusting client-side price IDs — verify on server
- Immediate cancellation instead of cancel_at_period_end — users lose access immediately
- Not handling failed payments — users keep access indefinitely
- Processing webhooks without signature verification — anyone could send fake events
- Not using idempotency keys — duplicate charges or duplicate access grants
- Hardcoding price IDs — use environment variables
- Creating a new customer for every checkout — reuse existing customers
- Not testing webhook failure scenarios — retries can cause duplicate processing