---
name: document-api
description: Generate API documentation from code including endpoints, parameters, types, and examples
---

## What I do

I generate clear API documentation from source code:

- Extract endpoint definitions, parameters, and return types
- Document error codes and response formats
- Add usage examples
- Follow existing documentation patterns in the project

## When to use me

Use this skill when:
- Adding or changing API endpoints
- Generating docs for an existing undocumented API
- Creating onboarding documentation for new team members
- Before publishing or versioning an API

## How I work

1. **Find the API surface** — Locate route definitions, handler functions, controller methods, or RPC definitions.
2. **Extract signatures** — For each endpoint: method/path, request parameters, request body schema, response schema, error codes.
3. **Trace types** — Follow type definitions to document request/response shapes accurately.
4. **Identify auth requirements** — Note which endpoints require authentication and what kind.
5. **Write documentation** — Follow the project's existing doc format (OpenAPI/Swagger, JSDoc, TSDoc, markdown, etc.).
6. **Add examples** — Include request/response examples for common use cases.

## Output format

Adapt to project conventions. Default markdown format:

```markdown
### `METHOD /path`

Description of what this endpoint does.

**Auth**: Required / None / Optional

**Parameters**:
| Name | Location | Type | Required | Description |
|------|----------|------|----------|-------------|

**Request Body**:
```json
{ "field": "value" }
```

**Response** `200`:
```json
{ "field": "value" }
```

**Errors**: `400`, `401`, `403`, `404`, `500`
```

## Guidelines

- Document from code, not from memory — verify each detail against the source
- Include all error codes the handler can return
- Examples should use realistic, not placeholder, values
- Match the style of existing documentation in the project