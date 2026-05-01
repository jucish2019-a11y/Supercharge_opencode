---
name: type-safety
description: Fix type errors, improve type coverage, and enforce stricter type checking
---

## What I do

I fix type errors, improve type coverage, and enforce stricter type checking in TypeScript codebases:

- Fix compilation errors and `any` types
- Design type architectures with generics, discriminated unions, and branded types
- Incrementally adopt strict mode in existing projects
- Strengthen types to catch runtime errors at compile time

## When to use me

Use this skill when:
- TypeScript is reporting type errors that need fixing
- A codebase has too many `any` types or `@ts-ignore` comments
- You want to enable stricter TypeScript options (strictNullChecks, noUncheckedIndexedAccess)
- Runtime errors could have been prevented with better types
- You need to design types for a new feature or API

## How I work

### Checker mode (auditing type safety)

1. **Run the type checker** — `tsc --noEmit` or the project's typecheck command. Capture all errors.
2. **Measure type coverage** — Count `any`, `as unknown as`, `@ts-ignore`, `@ts-expect-error`, and non-null assertions (`!`).
3. **Check tsconfig strictness** — Is `strict` enabled? `strictNullChecks`? `noUncheckedIndexedAccess`?
4. **Find unsafe patterns** — Type assertions (`as Type`), non-null assertions (`!`), empty catch blocks, untyped event handlers.
5. **Rate type coverage** — What percentage of the codebase has explicit types vs implicit `any`?

### Applier mode (fixing type issues)

1. **Start with the config** — Enable one strict option at a time. Fix all errors before enabling the next.
2. **Fix errors bottom-up** — Start with leaf modules (utilities, types, models) that have no internal dependencies, then work outward.
3. **Replace `any` with specific types** — Never use `any` when a more specific type is possible.
4. **Add runtime validation at boundaries** — API inputs, form submissions, external data. Use Zod or similar schema validation.
5. **Verify** — Run typecheck after each change. Ensure no new errors introduced.

## Enabling strict options incrementally

Don't enable all strict options at once. Enable one, fix all errors, then enable the next:

### Phase 1: `strictNullChecks`

The highest-impact strict option. Catches null/undefined runtime errors at compile time.

```json
// tsconfig.json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

Common fixes:

```typescript
// Before: assumes element exists
const el = document.getElementById('app');
el.textContent = 'Hello'; // Error: el could be null

// After: handle null case
const el = document.getElementById('app');
if (el) {
  el.textContent = 'Hello';
}

// Before: optional property accessed without check
function greet(user: { name?: string }) {
  return `Hello, ${user.name.toUpperCase()}`; // Error: name could be undefined
}

// After: provide default or check
function greet(user: { name?: string }) {
  return `Hello, ${(user.name ?? 'stranger').toUpperCase()}`;
}
```

### Phase 2: `noUncheckedIndexedAccess`

Catches undefined from array/object access:

```json
{
  "compilerOptions": {
    "noUncheckedIndexedAccess": true
  }
}
```

```typescript
// Before: assumes array element exists
const first = items[0].name; // Error: items[0] could be undefined

// After: handle undefined
const first = items[0]?.name ?? 'default';
// or
const [first] = items;
if (first) { first.name; }
```

### Phase 3: `noImplicitAny`

Prevents implicit any types on function parameters and return values:

```json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```

```typescript
// Before: parameter implicitly has 'any'
function process(data) { // Error: data implicitly has 'any'
  return data.map(item => item.value);
}

// After: explicit type
function process(data: DataItem[]): number[] {
  return data.map(item => item.value);
}
```

## Eliminating `any`

Replace `any` with specific types. Common patterns:

### Replace `any` with `unknown` when type is truly unknown

```typescript
// Before
function parse(json: string): any {
  return JSON.parse(json);
}

// After
function parse(json: string): unknown {
  return JSON.parse(json);
}

// Then narrow with type guards
const data = parse(response);
if (typeof data === 'object' && data !== null && 'name' in data) {
  console.log((data as { name: string }).name);
}
```

### Use generics instead of `any`

```typescript
// Before
function first(arr: any[]): any {
  return arr[0];
}

// After
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
```

### Use discriminated unions for variations

```typescript
// Before
type Action = {
  type: string;
  payload: any;
};

// After
type Action =
  | { type: 'ADD_TODO'; payload: { text: string } }
  | { type: 'DELETE_TODO'; payload: { id: string } }
  | { type: 'TOGGLE_TODO'; payload: { id: string } };

function handleAction(action: Action) {
  switch (action.type) {
    case 'ADD_TODO':
      // TypeScript knows payload is { text: string }
      break;
    case 'DELETE_TODO':
      // TypeScript knows payload is { id: string }
      break;
  }
}
```

### Use branded types for domain primitives

```typescript
// Prevent mixing up IDs that are both strings
type UserId = string & { __brand: 'UserId' };
type OrderId = string & { __brand: 'OrderId' };

function UserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId) { /* ... */ }
function getOrder(id: OrderId) { /* ... */ }

const uid = UserId('abc123');
getUser(uid); // OK
getOrder(uid); // Error: UserId is not assignable to OrderId
```

## Runtime validation at boundaries

TypeScript types are erased at runtime. Validate external data:

```typescript
import { z } from 'zod';

// Define the schema — this is both the runtime validator AND the TypeScript type
const UserSchema = z.object({
  id: z.string().ulid(),
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

type User = z.infer<typeof UserSchema>; // TypeScript type from the schema

// Use at API boundaries
function handleRequest(body: unknown): User {
  return UserSchema.parse(body); // Throws if invalid
}

// Safe parsing (doesn't throw)
function safeHandle(body: unknown) {
  const result = UserSchema.safeParse(body);
  if (!result.success) {
    return { error: result.error.flatten() };
  }
  return { data: result.data };
}
```

## Fixing common type errors

### "Type 'X' is not assignable to type 'Y'"

```typescript
// Problem: object literal inferred as broader type
const config = { host: 'localhost', port: 3000 }; // inferred as { host: string; port: number }

// Fix 1: Use const assertion
const config = { host: 'localhost', port: 3000 } as const; // { readonly host: 'localhost'; readonly port: 3000 }

// Fix 2: Explicit type
interface Config { host: string; port: number }
const config: Config = { host: 'localhost', port: 3000 };

// Fix 3: Satisfies (TypeScript 4.9+) — validates type without widening
const config = { host: 'localhost', port: 3000 } satisfies Config;
```

### "Property does not exist on type 'X'"

```typescript
// Problem: accessing a property that TypeScript doesn't know about
function getProp(obj: Record<string, unknown>, key: string) {
  return obj[key]; // unknown — correct, because we don't know if key exists
}

// Fix: narrow the type
function getProp<T extends string>(obj: Record<string, unknown>, key: T): T extends keyof typeof obj ? typeof obj[T] : undefined {
  return obj[key] as any; // use type assertion at the boundary only
}
```

### "Object is possibly 'null' or 'undefined'"

```typescript
// Fix 1: Null check
if (value !== null) { /* TypeScript narrows type */ }

// Fix 2: Optional chaining
value?.method();

// Fix 3: Nullish coalescing
const result = value ?? defaultValue;

// Fix 4: Non-null assertion (use sparingly — only when you know it's not null)
const el = document.getElementById('app')!;
```

## Type safety checklist

- [ ] `strict` mode enabled in tsconfig (or strict options enabled one by one)
- [ ] No `any` types in application code (test files can have explicit `any` for mocks)
- [ ] No `@ts-ignore` comments (use `@ts-expect-error` with a note instead)
- [ ] All function parameters have explicit types (no implicit `any`)
- [ ] All API boundaries use runtime validation (Zod, io-ts, or similar)
- [ ] No non-null assertions (`!`) without a safety comment explaining why it's safe
- [ ] All `as` type assertions have a comment explaining why the assertion is safe
- [ ] Generic types are used instead of `any` for reusable functions
- [ ] Discriminated unions used for state machines and variant types
- [ ] Index signatures validated with `noUncheckedIndexedAccess`

## Anti-patterns I avoid

- Using `any` as a type — use `unknown` if the type is truly unknown, or a specific type
- Using `as unknown as T` to bypass the type checker — fix the types instead
- Using `@ts-ignore` — use `@ts-expect-error` so TypeScript alerts you when the error is fixed
- Non-null assertions (`!`) without a safety comment — they silence the compiler but don't prevent runtime errors
- Overly verbose type guards that duplicate the type definition — use Zod schemas instead
- Defining the same type in multiple places — use shared type files and import
- Using `enum` — use `const` object patterns or string literal unions instead
- Creating type hierarchies that mirror runtime hierarchies — types should model data shapes, not class hierarchies