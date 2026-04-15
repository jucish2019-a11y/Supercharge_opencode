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

## Guidelines

- Never catch and swallow errors silently — at minimum, log them
- Always include the original cause (error chaining) when re-throwing
- Use operational errors for expected failures; let programmer errors crash
- Retry only idempotent operations
- Return meaningful error codes, not generic 500s
- Validate input at the boundary (API handler, CLI arg parser), not deep in business logic