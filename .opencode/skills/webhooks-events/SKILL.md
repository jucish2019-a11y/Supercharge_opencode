---
name: webhooks-events
description: Implement webhook receivers, event-driven architecture, message queues, retry logic, idempotency, and async workflows
---

## What I do

I implement webhook and event-driven systems:

- **Webhook receivers** — Signature verification, payload parsing, idempotent processing
- **Webhook senders** — Retry logic, exponential backoff, event type design
- **Message queues** — Bull/BullMQ, AWS SQS, Redis-based queues
- **Event-driven architecture** — Event sourcing, event buses, CQRS basics
- **Reliability** — Idempotency keys, dead letter queues, ordering guarantees
- **Observability** — Event tracing, delivery monitoring, failure alerts

## When to use me

Use this skill when:
- Receiving webhooks from Stripe, GitHub, Twilio, or other services
- Building a webhook API for external consumers
- Setting up background job processing
- Implementing event-driven communication between services
- Designing retry and failure handling for async operations
- Ensuring exactly-once or at-least-once delivery semantics

## Webhook receiver

### Signature verification

```ts
import crypto from 'crypto';

function verifyWebhookSignature(
  payload: string,
  signature: string,
  secret: string,
  algorithm: 'sha256' | 'sha512' = 'sha256'
): boolean {
  const expected = crypto
    .createHmac(algorithm, secret)
    .update(payload, 'utf8')
    .digest('hex');

  // Timing-safe comparison to prevent timing attacks
  try {
    return crypto.timingSafeEqual(
      Buffer.from(signature, 'hex'),
      Buffer.from(expected, 'hex')
    );
  } catch {
    return false;
  }
}

// Express/Next.js handler
export async function POST(req: Request) {
  const payload = await req.text();
  const signature = req.headers.get('x-webhook-signature')!;

  if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET!)) {
    return new Response('Invalid signature', { status: 401 });
  }

  const event = JSON.parse(payload);

  // Process idempotently
  await processWebhookEvent(event);

  return new Response('OK', { status: 200 });
}
```

### Idempotent processing

```ts
interface WebhookEvent {
  id: string;
  type: string;
  data: unknown;
  timestamp: string;
}

async function processWebhookEvent(event: WebhookEvent) {
  // Check if already processed
  const existing = await db.webhookEvent.findUnique({
    where: { id: event.id },
  });

  if (existing) {
    console.log(`Event ${event.id} already processed, skipping`);
    return;
  }

  // Process within a transaction
  await db.$transaction(async (tx) => {
    await tx.webhookEvent.create({
      data: {
        id: event.id,
        type: event.type,
        payload: event.data,
        processedAt: new Date(),
      },
    });

    // Handle specific event types
    switch (event.type) {
      case 'payment.succeeded':
        await handlePaymentSucceeded(event.data as PaymentData);
        break;
      case 'user.created':
        await handleUserCreated(event.data as UserData);
        break;
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }
  });
}
```

## Webhook sender

### Sending with retry

```ts
interface OutboundWebhook {
  id: string;
  url: string;
  payload: unknown;
  retries: number;
  maxRetries: number;
}

async function sendWebhook(webhook: OutboundWebhook): Promise<void> {
  const payload = JSON.stringify(webhook.payload);
  const signature = crypto
    .createHmac('sha256', process.env.WEBHOOK_SIGNING_SECRET!)
    .update(payload)
    .digest('hex');

  try {
    const response = await fetch(webhook.url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Webhook-Signature': signature,
        'X-Webhook-ID': webhook.id,
        'X-Webhook-Attempt': String(webhook.retries + 1),
      },
      body: payload,
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${await response.text()}`);
    }

    // Success: mark as delivered
    await db.webhookDelivery.update({
      where: { id: webhook.id },
      data: { status: 'DELIVERED', deliveredAt: new Date() },
    });
  } catch (error) {
    if (webhook.retries < webhook.maxRetries) {
      // Schedule retry with exponential backoff
      const delayMs = Math.min(
        Math.pow(2, webhook.retries) * 1000 + Math.random() * 1000,
        86400000 // Max 24 hours
      );

      await scheduleRetry(webhook.id, delayMs);
    } else {
      // Max retries exceeded: move to dead letter queue
      await moveToDeadLetterQueue(webhook.id, error);
    }
  }
}
```

## Message queues

### BullMQ (Redis-based)

```ts
import { Queue, Worker, Job } from 'bullmq';

// Define queue
const emailQueue = new Queue('emails', {
  connection: { host: 'localhost', port: 6379 },
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 },
    removeOnComplete: 100,
    removeOnFail: 50,
  },
});

// Add job
await emailQueue.add('send-welcome', {
  to: user.email,
  name: user.name,
}, {
  delay: 5000, // Delay 5 seconds
  priority: 1,
});

// Process jobs
const worker = new Worker('emails', async (job: Job) => {
  switch (job.name) {
    case 'send-welcome':
      await sendWelcomeEmail(job.data);
      break;
    case 'send-password-reset':
      await sendPasswordResetEmail(job.data);
      break;
  }
}, {
  connection: { host: 'localhost', port: 6379 },
  concurrency: 5,
});

// Error handling
worker.on('failed', (job, error) => {
  console.error(`Job ${job?.id} failed:`, error);
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  await worker.close();
  await emailQueue.close();
});
```

## Event-driven architecture patterns

### Event bus

```ts
interface DomainEvent {
  id: string;
  type: string;
  aggregateId: string;
  payload: unknown;
  timestamp: Date;
  version: number;
}

type EventHandler = (event: DomainEvent) => Promise<void>;

class EventBus {
  private handlers = new Map<string, EventHandler[]>();

  on(eventType: string, handler: EventHandler) {
    const existing = this.handlers.get(eventType) ?? [];
    existing.push(handler);
    this.handlers.set(eventType, existing);
  }

  async emit(event: DomainEvent) {
    const handlers = this.handlers.get(event.type) ?? [];
    await Promise.all(handlers.map(h => h(event).catch(err => {
      console.error(`Handler failed for event ${event.id}:`, err);
    })));
  }
}

// Usage
const eventBus = new EventBus();

eventBus.on('order.placed', async (event) => {
  await sendOrderConfirmationEmail(event.payload);
});

eventBus.on('order.placed', async (event) => {
  await updateInventory(event.payload);
});

eventBus.on('order.placed', async (event) => {
  await notifyWarehouse(event.payload);
});

// Emit
await eventBus.emit({
  id: crypto.randomUUID(),
  type: 'order.placed',
  aggregateId: order.id,
  payload: order,
  timestamp: new Date(),
  version: 1,
});
```

## Dead letter queues

```ts
interface DeadLetterEvent {
  originalEvent: WebhookEvent;
  error: string;
  failedAt: Date;
  retryCount: number;
}

async function moveToDeadLetterQueue(
  eventId: string,
  error: Error
): Promise<void> {
  const event = await db.webhookEvent.findUnique({ where: { id: eventId } });

  await db.deadLetterQueue.create({
    data: {
      originalEventId: eventId,
      payload: event?.payload,
      error: error.message,
      failedAt: new Date(),
      retryCount: event?.retryCount ?? 0,
    },
  });

  // Alert on-call
  await alertOnCall(`Webhook ${eventId} failed permanently: ${error.message}`);
}

// Manual retry from DLQ
async function retryDeadLetter(eventId: string): Promise<void> {
  const dlqItem = await db.deadLetterQueue.findUnique({ where: { id: eventId } });
  if (!dlqItem) throw new Error('Event not found in DLQ');

  await processWebhookEvent(dlqItem.payload as WebhookEvent);
  await db.deadLetterQueue.delete({ where: { id: eventId } });
}
```

## Quality checklist

- [ ] Webhook signatures verified with timing-safe comparison
- [ ] Idempotency keys prevent duplicate processing
- [ ] Payloads parsed safely (validate before processing)
- [ ] Retry logic with exponential backoff and jitter
- [ ] Dead letter queue for permanently failed events
- [ ] Graceful shutdown of queue workers
- [ ] Event schema versioning for backward compatibility
- [ ] Monitoring and alerting for queue depth and failures
- [ ] Ordering guarantees documented (at-least-once vs exactly-once)
- [ ] Webhook endpoints return 200 quickly — process async

## Anti-patterns I avoid

- Processing webhooks synchronously in the HTTP handler — always queue
- Not verifying webhook signatures — anyone can send fake events
- No idempotency — duplicate events cause duplicate actions
- Infinite retry without backoff or max retry limit
- Not handling webhook payload validation — malformed data crashes the handler
- Losing events on service restart — persistent queues required
- Tight coupling between event producer and consumer — use event bus
- Not monitoring queue depth — silent failures accumulate