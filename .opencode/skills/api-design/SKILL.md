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