---
name: backend-api
description: Build Node.js backend APIs — framework selection, middleware, validation, error handling, rate limiting, and production patterns for Express, Fastify, and Hono
---

## What I do

I build backend APIs in Node.js/TypeScript:

- **Framework selection** — Express, Fastify, Hono — when to use each
- **Route structure** — RESTful resource routing, versioning, organization
- **Middleware** — Auth, validation, rate limiting, logging, CORS
- **Request validation** — Zod schemas, input sanitization, type-safe handlers
- **Error handling** — Structured errors, global error middleware, status codes
- **Response patterns** — Pagination, filtering, consistent envelope
- **Background jobs** — Queues, scheduled tasks, webhooks

## When to use me

Use this skill when:
- Building a new API backend in Node.js
- Choosing between Express, Fastify, and Hono
- Structuring routes, middleware, and error handling
- Adding validation, rate limiting, or authentication to API routes
- Implementing pagination, filtering, and sorting
- Setting up background jobs or webhooks

## Framework selection

| Framework | Use when |Strengths | Weaknesses |
|-----------|----------|----------|------------|
| **Hono** | Edge/serverless, small APIs, Next.js API routes | Ultra-light, multi-runtime, TypeScript-first | Smaller ecosystem |
| **Fastify** | High-throughput APIs, microservices | Fastest benchmarks, schema validation built-in | Verbose plugin system |
| **Express** | Existing team knowledge, large middleware ecosystem | Most familiar, most tutorials | Slowest, no TypeScript-first design |

Default recommendation: **Hono** for new projects (fast, lightweight, edge-compatible). **Fastify** for high-throughput server APIs. **Express** only if the team requires it.

## Project structure

```
src/
  index.ts                 ← Entry point, server setup
  routes/
    index.ts               ← Route registration
    projects.ts            ← Project routes
    tasks.ts               ← Task routes
    auth.ts                ← Auth routes
  middleware/
    auth.ts                ← Authentication middleware
    rate-limit.ts          ← Rate limiting
    validate.ts            ← Request validation
  lib/
    db.ts                  ← Database client
    errors.ts              ← Custom error classes
    utils.ts               ← Utility functions
  schemas/
    project.ts             ← Zod schemas for project endpoints
    task.ts                ← Zod schemas for task endpoints
  types/
    index.ts               ← Shared types
```

## Type-safe API with Hono + Zod

```ts
import { Hono } from 'hono';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';

const app = new Hono();

// Schemas
const CreateProjectSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(1000).optional(),
  isPublic: z.boolean().default(false),
});

const ProjectParamsSchema = z.object({
  id: z.string().uuid(),
});

// Routes
app.post('/api/projects',
  zValidator('json', CreateProjectSchema),
  async (c) => {
    const data = c.req.valid('json'); // Fully typed!
    const project = await db.project.create({ data });
    return c.json({ data: project }, 201);
  }
);

app.get('/api/projects/:id',
  zValidator('param', ProjectParamsSchema),
  async (c) => {
    const { id } = c.req.valid('param');
    const project = await db.project.findUnique({ where: { id } });
    if (!project) throw new NotFoundError('Project not found');
    return c.json({ data: project });
  }
);
```

## RESTful route patterns

```
Resource: Projects

GET    /api/projects           → List projects (paginated, filterable)
POST   /api/projects           → Create project
GET    /api/projects/:id       → Get single project
PATCH  /api/projects/:id       → Update project
DELETE /api/projects/:id       → Delete project

Nested: Project Tasks

GET    /api/projects/:id/tasks → List tasks in project
POST   /api/projects/:id/tasks → Add task to project

Actions (non-CRUD):

POST   /api/projects/:id/archive   → Archive project
POST   /api/projects/:id/transfer   → Transfer ownership
```

### Naming rules
- Resources are plural nouns: `/projects` not `/project`
- Nested resources under parent: `/projects/:id/tasks`
- Actions are verbs: `/projects/:id/archive`
- Use PATCH (not PUT) for partial updates
- Use POST for actions that aren't CRUD

## Response envelope

```ts
// Success response
{
  "data": { ... }              // Single resource
}

// Success response with pagination
{
  "data": [ ... ],             // Array of resources
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 143,
    "totalPages": 8
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Name must be at least 1 character",
    "details": {
      "name": ["Required", "Must be at least 1 character"]
    }
  }
}
```

## Pagination

```ts
// Query schema
const PaginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  pageSize: z.coerce.number().int().min(1).max(100).default(20),
  sortBy: z.string().default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
});

// Handler
app.get('/api/projects', async (c) => {
  const { page, pageSize, sortBy, sortOrder } = c.req.valid('query');

  const [data, totalItems] = await Promise.all([
    db.project.findMany({
      skip: (page - 1) * pageSize,
      take: pageSize,
      orderBy: { [sortBy]: sortOrder },
    }),
    db.project.count(),
  ]);

  return c.json({
    data,
    pagination: {
      page,
      pageSize,
      totalItems,
      totalPages: Math.ceil(totalItems / pageSize),
    },
  });
});
```

Cursor-based pagination for large datasets:

```ts
app.get('/api/activities', async (c) => {
  const { cursor, limit = 20 } = c.req.valid('query');

  const data = await db.activity.findMany({
    take: limit + 1,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' },
  });

  const hasMore = data.length > limit;
  const items = hasMore ? data.slice(0, -1) : data;

  return c.json({
    data: items,
    nextCursor: hasMore ? items[items.length - 1].id : null,
  });
});
```

## Error handling

```ts
// Custom error classes
class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number,
    public details?: Record<string, string[]>
  ) {
    super(message);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super('NOT_FOUND', `${resource} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(details: Record<string, string[]>) {
    super('VALIDATION_ERROR', 'Validation failed', 400, details);
  }
}

class AuthError extends AppError {
  constructor(message = 'Unauthorized') {
    super('UNAUTHORIZED', message, 401);
  }
}

class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super('FORBIDDEN', message, 403);
  }
}

class RateLimitError extends AppError {
  constructor() {
    super('RATE_LIMITED', 'Too many requests', 429);
  }
}

// Global error handler
app.onError((err, c) => {
  if (err instanceof AppError) {
    return c.json({ error: { code: err.code, message: err.message, details: err.details } }, err.statusCode);
  }

  // Unexpected errors — log but don't expose internals
  console.error('Unhandled error:', err);
  return c.json({ error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } }, 500);
});
```

## Middleware patterns

### Authentication

```ts
import { verify } from 'jsonwebtoken';

const authMiddleware = async (c, next) => {
  const token = c.req.header('Authorization')?.replace('Bearer ', '');
  if (!token) throw new AuthError();

  try {
    const payload = verify(token, process.env.JWT_SECRET);
    c.set('userId', payload.sub);
    await next();
  } catch {
    throw new AuthError('Invalid or expired token');
  }
};

// Apply to protected routes
app.use('/api/*', authMiddleware);
```

### Rate limiting

```ts
import { rateLimiter } from 'hono-rate-limiter';

// Global rate limit
app.use(rateLimiter({
  windowMs: 60 * 1000,
  max: 100,
}));

// Strict rate limit for auth
app.use('/api/auth/*', rateLimiter({
  windowMs: 60 * 1000,
  max: 5,
  keyGenerator: (c) => c.req.header('x-forwarded-for') ?? 'unknown',
}));
```

### CORS

```ts
import { cors } from 'hono/cors';

app.use('*', cors({
  origin: (origin) => {
    const allowed = ['https://app.example.com', 'https://admin.example.com'];
    return allowed.includes(origin) ? origin : '';
  },
  allowMethods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400,
}));
```

## Background jobs

### Job queue pattern

```ts
// For production: use BullMQ + Redis
// For simple apps: use a job table in the database

// Job definition
type JobType = 'send_email' | 'process_export' | 'sync_webhook';

interface Job<T = unknown> {
  id: string;
  type: JobType;
  payload: T;
  status: 'pending' | 'processing' | 'completed' | 'failed';
  attempts: number;
  maxAttempts: number;
  createdAt: Date;
  processAfter: Date;
}

// Job processor
async function processJob(job: Job) {
  switch (job.type) {
    case 'send_email':
      await sendEmail(job.payload);
      break;
    case 'process_export':
      await generateExport(job.payload);
      break;
    case 'sync_webhook':
      await deliverWebhook(job.payload);
      break;
  }
}
```

## API quality checklist

- [ ] All inputs validated with Zod schemas (never trust the client)
- [ ] Consistent response envelope (data/error structure)
- [ ] Pagination on all list endpoints
- [ ] Cursor pagination for large/streaming datasets
- [ ] Proper status codes (201, 204, 400, 401, 403, 404, 429, 500)
- [ ] Rate limiting on all endpoints (stricter on auth)
- [ ] CORS configured for allowed origins only
- [ ] Global error handler catches AppError and unexpected errors
- [ ] Request ID in response headers for tracing
- [ ] No stack traces or internals in error responses
- [ ] Timeout on all external calls
- [ ] Idempotency keys on write endpoints

## Anti-patterns I avoid

- Trusting client input without validation — always validate with Zod
- Returning raw database objects — control what the API exposes
- Using 200 for everything — use proper HTTP status codes
- Exposing stack traces in error responses — security risk
- No rate limiting — APIs get abused without it
- Synchronous processing for long tasks — use background jobs
- PUT for partial updates — use PATCH
- No pagination on list endpoints — unbounded queries kill performance
- Hardcoded page sizes — make them configurable with a maximum