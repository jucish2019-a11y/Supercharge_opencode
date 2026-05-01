---
name: api-design
description: Design new REST or GraphQL APIs from requirements with consistent patterns and conventions
---

## What I do

I design APIs from requirements:

- Define resource models and endpoint structure
- Choose consistent naming, pagination, filtering, and error patterns
- Design request/response schemas with proper types
- Plan versioning and backward compatibility

## When to use me

Use this skill when:
- Designing a new API or service from scratch
- Adding significant new endpoints to an existing API
- Re-designing an API that has grown inconsistently
- Planning a public API that will be consumed by external clients

## How I work

1. **Gather requirements** — Identify resources, actions, and relationships. List all operations the API must support. Determine who the consumers are.
2. **Audit existing patterns** — If extending an existing API, find the conventions: URL structure, auth method, pagination style, error format, naming conventions. Stay consistent.
3. **Design the resource model** — Define the core entities and their relationships. Choose singular vs plural resource names. Determine what's a resource and what's a sub-resource.
4. **Define endpoints** — Map operations to HTTP methods:
   - `GET /resources` — List (with pagination, filtering, sorting)
   - `GET /resources/:id` — Get one
   - `POST /resources` — Create
   - `PUT /resources/:id` — Full replace
   - `PATCH /resources/:id` — Partial update
   - `DELETE /resources/:id` — Delete
5. **Design request/response schemas** — Define input validation rules, output shapes, and status codes. Use consistent field naming (camelCase or snake_case throughout).
6. **Plan error responses** — Consistent error shape: `{ error: { code, message, details } }`. Map common scenarios to status codes.
7. **Plan versioning** — Choose a strategy (URL path `/v1/`, headers, or content negotiation). Stick with it.
8. **Document** — Write an OpenAPI spec or equivalent. Include examples for every endpoint.

## OpenAPI specification

```yaml
openapi: 3.0.0
info:
  title: Example API
  version: 1.0.0
  description: A well-designed REST API example

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging

paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
        - name: sort
          in: query
          schema:
            type: string
            enum: [createdAt, name, email]
        - name: order
          in: query
          schema:
            type: string
            enum: [asc, desc]
            default: desc
      responses:
        200:
          description: List of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'

    post:
      summary: Create user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserInput'
      responses:
        201:
          description: User created
          headers:
            Location:
              schema:
                type: string
              description: URL of the created user
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        422:
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ValidationError'

  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        200:
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        404:
          description: User not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        role:
          type: string
          enum: [user, admin]
        createdAt:
          type: string
          format: date-time
      required: [id, email, name, role, createdAt]

    CreateUserInput:
      type: object
      properties:
        email:
          type: string
          format: email
        name:
          type: string
          minLength: 1
          maxLength: 100
        password:
          type: string
          minLength: 8
      required: [email, name, password]

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
        hasNext:
          type: boolean
        hasPrev:
          type: boolean

    Error:
      type: object
      properties:
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            statusCode:
              type: integer

    ValidationError:
      allOf:
        - $ref: '#/components/schemas/Error'
        - type: object
          properties:
            error:
              type: object
              properties:
                fields:
                  type: array
                  items:
                    type: object
                    properties:
                      path:
                        type: string
                      message:
                        type: string
```

## Pagination implementation

### Offset-based

```ts
app.get('/api/users', async (req, res) => {
  const page = Math.max(1, parseInt(req.query.page as string) || 1);
  const limit = Math.min(100, parseInt(req.query.limit as string) || 20);
  const offset = (page - 1) * limit;

  const [users, total] = await Promise.all([
    db.user.findMany({
      skip: offset,
      take: limit,
      orderBy: { createdAt: 'desc' },
    }),
    db.user.count(),
  ]);

  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
      hasNext: page < Math.ceil(total / limit),
      hasPrev: page > 1,
    },
  });
});
```

### Cursor-based

```ts
app.get('/api/users', async (req, res) => {
  const limit = Math.min(100, parseInt(req.query.limit as string) || 20);
  const cursor = req.query.cursor as string | undefined;

  const users = await db.user.findMany({
    take: limit + 1,
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' },
  });

  const hasNextPage = users.length > limit;
  const data = hasNextPage ? users.slice(0, -1) : users;

  res.json({
    data,
    pagination: {
      nextCursor: hasNextPage ? data[data.length - 1].id : null,
      hasNextPage,
    },
  });
});
```

## Error response format

```ts
interface ApiError {
  error: {
    code: string;
    message: string;
    statusCode: number;
    details?: Record<string, unknown>;
  };
}

// Standard error responses
const ERROR_RESPONSES = {
  BAD_REQUEST: { code: 'BAD_REQUEST', message: 'Invalid request', statusCode: 400 },
  UNAUTHORIZED: { code: 'UNAUTHORIZED', message: 'Authentication required', statusCode: 401 },
  FORBIDDEN: { code: 'FORBIDDEN', message: 'Access denied', statusCode: 403 },
  NOT_FOUND: { code: 'NOT_FOUND', message: 'Resource not found', statusCode: 404 },
  CONFLICT: { code: 'CONFLICT', message: 'Resource conflict', statusCode: 409 },
  VALIDATION_ERROR: { code: 'VALIDATION_ERROR', message: 'Validation failed', statusCode: 422 },
  RATE_LIMITED: { code: 'RATE_LIMITED', message: 'Too many requests', statusCode: 429 },
  INTERNAL_ERROR: { code: 'INTERNAL_ERROR', message: 'Internal server error', statusCode: 500 },
};

// Express error handler
app.use((error: Error, req: Request, res: Response, _next: NextFunction) => {
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      error: {
        code: error.code,
        message: error.message,
        statusCode: error.statusCode,
      },
    });
  }

  console.error('Unhandled error:', error);
  return res.status(500).json({
    error: ERROR_RESPONSES.INTERNAL_ERROR,
  });
});
```

## Rate limiting

```ts
import rateLimit from 'express-rate-limit';

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: {
        code: 'RATE_LIMITED',
        message: 'Too many requests, please try again later',
        statusCode: 429,
        retryAfter: Math.ceil(req.rateLimit.resetTime.getTime() / 1000),
      },
    });
  },
});

// Stricter limiter for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
});

app.use(globalLimiter);
app.use('/api/auth', authLimiter);
```

## CORS configuration

```ts
import cors from 'cors';

app.use(cors({
  origin: (origin, callback) => {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') ?? [];
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Request-ID'],
}));
```

## Versioning strategies

```ts
// URL path versioning (recommended for public APIs)
app.use('/v1/users', usersV1Router);
app.use('/v2/users', usersV2Router);

// Header versioning
app.use('/users', (req, res, next) => {
  const version = req.headers['api-version'] || '1';
  if (version === '2') {
    return usersV2Router(req, res, next);
  }
  return usersV1Router(req, res, next);
});

// Content negotiation
app.use('/users', (req, res, next) => {
  const acceptHeader = req.headers.accept || '';
  if (acceptHeader.includes('application/vnd.api.v2+json')) {
    return usersV2Router(req, res, next);
  }
  return usersV1Router(req, res, next);
});
```

## REST conventions

- Use plural nouns for collections: `/users`, `/orders`
- Nest for clear ownership: `/users/:id/orders`
- Use query params for filtering, sorting, pagination: `?status=active&sort=-created_at&page=2`
- Return appropriate status codes: 201 for create, 204 for delete, 422 for validation
- Use `PATCH` for partial updates, `PUT` for full replacement
- Include `Location` header for created resources

## Guidelines

- APIs are contracts — design for backward compatibility from day one
- Be consistent: naming, pagination, error format, auth headers
- Prefer coarse-grained endpoints over fine-grained chatty APIs
- Include pagination from the start — never return unbounded lists
- Validate input at the boundary, not in business logic
- Never expose internal IDs, implementation details, or stack traces

## Quality checklist

- [ ] OpenAPI spec exists and is up to date
- [ ] Pagination implemented for all list endpoints
- [ ] Consistent error response format across all endpoints
- [ ] Rate limiting configured for all endpoints
- [ ] Input validation at API boundary
- [ ] CORS properly configured
- [ ] Versioning strategy documented
- [ ] Authentication on all sensitive endpoints
- [ ] Request logging and tracing
- [ ] API documentation with examples