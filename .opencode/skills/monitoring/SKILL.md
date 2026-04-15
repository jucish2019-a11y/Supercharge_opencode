---
name: monitoring
description: Implement observability — structured logging, error tracking, application performance monitoring, alerting, and health checks for production systems
---

## What I do

I implement production observability so you know what's happening in your application:

- **Structured logging** — JSON logs, correlation IDs, log levels, redaction
- **Error tracking** — Sentry, error boundaries, source maps, error grouping
- **Application performance** — Web Vitals, server timing, slow query detection
- **Health checks** — Liveness, readiness, dependency checks
- **Alerting** — What to alert on, escalation, runbooks
- **Dashboards** — Key metrics, user-facing SLIs, infrastructure metrics

## When to use me

Use this skill when:
- Setting up logging, error tracking, or monitoring for a new project
- Configuring Sentry or similar error tracking
- Implementing health check endpoints
- Defining alerting rules and escalation
- Adding Web Vitals performance monitoring
- Debugging production issues without visibility

## The three pillars of observability

| Pillar | Tool | Answers |
|--------|------|---------|
| **Logs** | Structured logger (pino, winston) | What happened? (discrete events) |
| **Metrics** | Prometheus, DataDog, Vercel Analytics | How much? How fast? (aggregated numbers) |
| **Traces** | OpenTelemetry, Sentry Tracing | Where did it go? (request flow across services) |

## Structured logging

### Logger setup

```ts
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined,
  redact: ['password', 'token', 'authorization', 'cookie'],
  serializers: {
    err: pino.stdSerializers.err,
    req: (req) => ({
      method: req.method,
      url: req.url,
      headers: { 'user-agent': req.headers['user-agent'] },
    }),
  },
});

export default logger;
```

### Log levels

```
fatal:  App is unusable, immediate action required (DB down, data corruption)
error:  Request failed, but app continues (API 500, unhandled exception)
warn:   Unexpected but handled (deprecated API used, retry succeeded, slow query)
info:   Normal business events (user signed in, project created, payment processed)
debug:  Debugging information (query executed, cache hit/miss, middleware chain)
trace:  Very detailed (function entry/exit, variable values)
```

Rules:
- Production: `info` minimum
- Staging: `debug` minimum
- Development: `debug` or `trace`
- Never log at `trace` or `debug` in production — performance impact
- Never log sensitive data (passwords, tokens, PII) — use redaction

### Structured log format

```json
{
  "level": "info",
  "time": 1713196800000,
  "msg": "Project created",
  "userId": "user_123",
  "projectId": "proj_456",
  "requestId": "req_abc",
  "duration": 145,
  "statusCode": 201
}
```

Every log entry should include:
- `requestId` — correlates all logs from a single request
- `userId` — who triggered the event (if authenticated)
- Fields specific to the event (projectId, taskId, etc.)
- `duration` — how long the operation took (in ms)

### Request ID middleware

```ts
import { nanoid } from 'nanoid';

app.use('*', async (c, next) => {
  const requestId = c.req.header('x-request-id') || nanoid();
  c.set('requestId', requestId);
  c.header('x-request-id', requestId);

  logger.info({ requestId, method: c.req.method, url: c.req.url }, 'Request started');

  const start = Date.now();
  await next();
  const duration = Date.now() - start;

  logger.info({
    requestId,
    statusCode: c.res.status,
    duration,
  }, 'Request completed');
});
```

## Error tracking with Sentry

### Setup

```ts
// lib/sentry.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.NEXT_PUBLIC_VERSION,
  tracesSampleRate: 0.1,    // 10% of transactions
  replaysSessionSampleRate: 0.01,  // 1% of sessions
  replaysOnErrorSampleRate: 1.0,   // 100% on error
  beforeSend(event) {
    // Scrub PII
    if (event.request?.headers) {
      delete event.request.headers.authorization;
      delete event.request.headers.cookie;
    }
    return event;
  },
});
```

### Error reporting

```ts
try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { feature: 'project-creation' },
    user: { id: userId, email: userEmail },
    extra: { projectId, projectName },
  });
  throw error;
}
```

### React error boundary

```tsx
import * as Sentry from '@sentry/nextjs';
import { ErrorBoundary } from '@sentry/nextjs';

function CustomFallback({ error, resetError }) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={resetError}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary fallback={CustomFallback} showDialog>
      <YourApp />
    </ErrorBoundary>
  );
}
```

### Source maps

```
For Sentry to show readable stack traces:
  1. Build with source maps: next build (generates automatically)
  2. Upload to Sentry: sentry-cli sourcemaps upload
  3. Don't expose source maps to users (delete after upload or set hidden source map)

next.config.js:
  sentry: { hideSourceMaps: true }
```

## Web Vitals (client-side performance)

```tsx
// Core Web Vitals to track:
// LCP (Largest Contentful Paint) — loading speed
// FID (First Input Delay) — interactivity
// CLS (Cumulative Layout Shift) — visual stability
// INP (Interaction to Next Paint) — responsiveness (replaces FID)
// TTFB (Time to First Byte) — server responsiveness

import { useReportWebVitals } from 'next/web-vitals';

function WebVitalsReporter() {
  useReportWebVitals((metric) => {
    fetch('/api/vitals', {
      method: 'POST',
      body: JSON.stringify({
        name: metric.name,
        value: metric.value,
        rating: metric.rating,
        delta: metric.delta,
        id: metric.id,
        path: window.location.pathname,
      }),
    });
  });
  return null;
}
```

### Web Vitals targets

| Metric | Good | Needs improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤2.5s | 2.5-4.0s | >4.0s |
| INP | ≤200ms | 200-500ms | >500ms |
| CLS | ≤0.1 | 0.1-0.25 | >0.25 |
| TTFB | ≤800ms | 800-1800ms | >1800ms |

## Health checks

### Liveness check (is the process running?)

```ts
app.get('/api/health', (c) => {
  return c.json({ status: 'ok', timestamp: new Date().toISOString() });
});
```

### Readiness check (can it serve traffic?)

```ts
app.get('/api/health/ready', async (c) => {
  const checks: Record<string, { ok: boolean; latency?: number }> = {};

  // Database
  try {
    const start = Date.now();
    await db.$queryRaw`SELECT 1`;
    checks.database = { ok: true, latency: Date.now() - start };
  } catch {
    checks.database = { ok: false };
  }

  // Redis
  try {
    const start = Date.now();
    await redis.ping();
    checks.redis = { ok: true, latency: Date.now() - start };
  } catch {
    checks.redis = { ok: false };
  }

  const allOk = Object.values(checks).every(c => c.ok);
  return c.json({ status: allOk ? 'ok' : 'degraded', checks }, allOk ? 200 : 503);
});
```

## Alerting

### What to alert on

```
Critical (wake someone up):
  - Error rate > 5% (5xx responses)
  - P99 latency > 5s
  - Health check failing
  - Database connection pool exhausted
  - Disk > 90% full
  - Payment processing failures

Warning (notify during business hours):
  - Error rate > 1%
  - P99 latency > 2s
  - Memory usage > 80%
  - Slow query detected (> 1s)
  - Stale data (ISR revalidation failing)

Info (dashboard only, no notification):
  - Request volume changes
  - New deployment metrics
  - Feature flag changes
```

### Alert rules

```
1. Every alert has an owner (who gets paged?)
2. Every alert has a runbook (what do you do when it fires?)
3. Alert on symptoms, not causes (user impact, not internal metrics)
4. Use multiple thresholds (warning before critical)
5. Rate-limit alerts (don't spam — 1 alert per 5 minutes per rule)
6. Auto-resolve when conditions clear
```

## Dashboard metrics

### Application metrics

```
Request metrics:
  - Request rate (req/s per endpoint)
  - Error rate (% of 5xx responses)
  - P50, P95, P99 latency per endpoint
  - Active requests (concurrency)

Business metrics:
  - Signups per day
  - Active users per day
  - Projects created per day
  - Payments processed per day

Resource metrics:
  - CPU usage
  - Memory usage
  - Database connection pool usage
  - Cache hit rate
```

## Quality checklist

- [ ] Structured logging with pino or equivalent (not console.log)
- [ ] Request IDs on all requests (correlated logs)
- [ ] Sensitive data redacted from logs (passwords, tokens, PII)
- [ ] Sentry or equivalent for error tracking with source maps
- [ ] Error boundaries in React with user-friendly fallbacks
- [ ] Web Vitals tracked and reported
- [ ] Health check endpoints for liveness and readiness
- [ ] Dependency health checked (DB, Redis, external services)
- [ ] Alerts defined for critical conditions with owners and runbooks
- [ ] Dashboard shows error rate, latency, and business metrics
- [ ] Log level set appropriately per environment
- [ ] Production errors create Sentry issues, not console logs

## Anti-patterns I avoid

- console.log in production — use structured logging
- Logging full request bodies (may contain PII) — redact
- Alerting without a runbook — when the alert fires, no one knows what to do
- Alerting on every error — alert on rate spikes, not individual errors
- Health checks that always return OK — they should test real dependencies
- Monitoring without dashboards — metrics without visualization are unused
- Source maps in production without Sentry — users can read your source code
- Ignoring Web Vitals — they directly impact SEO and user experience