---
name: error-handling
description: Design robust error types, recovery strategies, and graceful degradation patterns
---

## What I do

I design and implement robust error handling:

- Create typed error hierarchies for the domain
- Implement recovery and fallback strategies
- Design graceful degradation for partial failures
- Add proper logging and observability for errors

## When to use me

Use this skill when:
- Adding error handling to a new module or service
- Replacing ad-hoc error handling with a consistent system
- Errors are silently swallowed or crash the app unexpectedly
- Building APIs that need structured error responses
- Handling distributed system failures (network, timeout, rate limit)

## How I work

1. **Audit current error handling** — Find where errors are caught, logged, swallowed, or re-thrown. Identify gaps: unhandled promise rejections, missing try/catch, bare `catch (e) {}` blocks.
2. **Define error types** — Create a hierarchy matching the domain:
   - Base error class with common fields (code, message, cause, timestamp)
   - Specific error classes per failure mode (NotFound, ValidationFailed, Unauthorized, etc.)
   - Distinguish between operational errors (expected, recoverable) and programmer errors (bugs, unrecoverable)
3. **Design recovery strategies** — For each operational error type:
   - **Retry**: For transient failures (network, timeout) — use exponential backoff with jitter
   - **Fallback**: Return cached/stale data, default value, or degraded feature
   - **Circuit breaker**: Stop trying a failing dependency for a cooldown period
   - **Fail fast**: For invalid state or programmer errors — crash or return immediately
4. **Implement error boundaries** — Place try/catch at strategic boundaries: API handlers, message consumers, UI components, middleware. Never catch silently without action.
5. **Structure error responses** — For APIs, return consistent error shapes: `{ error: { code, message, details } }`. Never expose internal details or stack traces to clients.
6. **Add observability** — Log errors with context (what operation, what inputs, what state). Use structured logging. Track error rates and types.

## Error type design pattern

```
AppError (base)
├── OperationalError
│   ├── NotFoundError
│   ├── ValidationError
│   ├── AuthenticationError
│   ├── AuthorizationError
│   ├── RateLimitError
│   └── ExternalServiceError
└── ProgrammerError
    └── AssertionError
```

## TypeScript error classes

```ts
// Base error class
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  readonly timestamp: Date;
  readonly cause?: Error;

  constructor(message: string, cause?: Error) {
    super(message);
    this.name = this.constructor.name;
    this.timestamp = new Date();
    this.cause = cause;
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      code: this.code,
      message: this.message,
      statusCode: this.statusCode,
      timestamp: this.timestamp.toISOString(),
    };
  }
}

// Operational errors (expected, recoverable)
class NotFoundError extends AppError {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;
}

class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 422;

  constructor(
    message: string,
    public readonly fields: Array<{ path: string; message: string }>,
    cause?: Error
  ) {
    super(message, cause);
  }

  override toJSON() {
    return {
      ...super.toJSON(),
      fields: this.fields,
    };
  }
}

class AuthenticationError extends AppError {
  readonly code = 'UNAUTHENTICATED';
  readonly statusCode = 401;
}

class AuthorizationError extends AppError {
  readonly code = 'FORBIDDEN';
  readonly statusCode = 403;
}

class RateLimitError extends AppError {
  readonly code = 'RATE_LIMITED';
  readonly statusCode = 429;

  constructor(
    message: string,
    public readonly retryAfter: number,
    cause?: Error
  ) {
    super(message, cause);
  }
}

class ExternalServiceError extends AppError {
  readonly code = 'EXTERNAL_SERVICE_ERROR';
  readonly statusCode = 502;
}

// Programmer errors (bugs, unrecoverable)
class AssertionError extends AppError {
  readonly code = 'ASSERTION_FAILED';
  readonly statusCode = 500;
}
```

## Retry and backoff

```ts
interface RetryOptions {
  maxRetries: number;
  baseDelay: number;
  maxDelay: number;
  retryableErrors: string[];
}

async function withRetry<T>(
  fn: () => Promise<T>,
  options: Partial<RetryOptions> = {}
): Promise<T> {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    maxDelay = 30000,
    retryableErrors = ['ETIMEDOUT', 'ECONNRESET', 'ENOTFOUND'],
  } = options;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error: any) {
      if (attempt === maxRetries) throw error;

      const isRetryable = retryableErrors.includes(error.code) ||
        error instanceof ExternalServiceError;

      if (!isRetryable) throw error;

      // Exponential backoff with jitter
      const delay = Math.min(
        baseDelay * Math.pow(2, attempt) + Math.random() * 1000,
        maxDelay
      );

      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw new Error('Unreachable');
}
```

## Circuit breaker

```ts
interface CircuitBreakerOptions {
  failureThreshold: number;
  resetTimeout: number;
}

type CircuitState = 'CLOSED' | 'OPEN' | 'HALF_OPEN';

class CircuitBreaker {
  private state: CircuitState = 'CLOSED';
  private failures = 0;
  private lastFailureTime?: number;

  constructor(
    private fn: () => Promise<any>,
    private options: CircuitBreakerOptions = { failureThreshold: 5, resetTimeout: 30000 }
  ) {}

  async execute(): Promise<any> {
    if (this.state === 'OPEN') {
      if (Date.now() - (this.lastFailureTime ?? 0) > this.options.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new ExternalServiceError('Circuit breaker is open');
      }
    }

    try {
      const result = await this.fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();

    if (this.failures >= this.options.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}
```

## React error boundaries

```tsx
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div role="alert" className="p-4 bg-red-50 border border-red-200 rounded">
          <h2 className="text-lg font-semibold text-red-800">Something went wrong</h2>
          <p className="text-red-600 mt-1">{this.state.error?.message}</p>
          <button
            onClick={() => this.setState({ hasError: false })}
            className="mt-3 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorPage />}>
  <App />
</ErrorBoundary>
```

## API error responses

```ts
// Express error handler
app.use((error: Error, req: Request, res: Response, _next: NextFunction) => {
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      error: error.toJSON(),
    });
  }

  // Unknown error — don't leak details
  console.error('Unhandled error:', error);
  return res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
      statusCode: 500,
    },
  });
});
```

## Structured logging

```ts
interface LogContext {
  operation: string;
  userId?: string;
  requestId: string;
  [key: string]: any;
}

function logError(error: Error, context: LogContext) {
  const logEntry = {
    level: 'error',
    timestamp: new Date().toISOString(),
    message: error.message,
    stack: error.stack,
    ...context,
  };

  // Send to logging service
  console.error(JSON.stringify(logEntry));
}

// Usage
try {
  await processOrder(orderId);
} catch (error) {
  logError(error as Error, {
    operation: 'processOrder',
    orderId,
    userId: req.user?.id,
    requestId: req.id,
  });
}
```

## Guidelines

- Never catch and swallow errors silently — at minimum, log them
- Always include the original cause (error chaining) when re-throwing
- Use operational errors for expected failures; let programmer errors crash
- Retry only idempotent operations
- Return meaningful error codes, not generic 500s
- Validate input at the boundary (API handler, CLI arg parser), not deep in business logic

## Quality checklist

- [ ] Custom error types for each domain failure mode
- [ ] Operational vs programmer errors clearly distinguished
- [ ] Retry logic with exponential backoff for transient failures
- [ ] Circuit breaker for external service calls
- [ ] Error boundaries at route/page level in React
- [ ] Consistent API error response format
- [ ] Structured logging with request context
- [ ] Stack traces hidden from API clients
- [ ] Error monitoring integration (Sentry, etc.)
- [ ] Fail-fast for invalid state or programmer errors