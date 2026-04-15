---
name: typescript
description: Design TypeScript type architectures — generics, utility types, declaration files, discriminated unions, brand types, and type-level programming for bulletproof code
---

## What I do

I design TypeScript type systems that eliminate entire classes of bugs at compile time:

- **Type architecture** — How to structure types across a project (shared, feature, component)
- **Generics** — Reusable type constructors with constraints and defaults
- **Utility types** — Built-in and custom type transformers (Pick, Omit, Record, etc.)
- **Discriminated unions** — Type-safe state machines and data modeling
- **Branded types** — Nominal typing for domain safety (UserId vs ProjectId)
- **Declaration files** — .d.ts for untyped dependencies and global types
- **Type narrowing** — Exhaustive checks, type guards, assertion functions

## When to use me

Use this skill when:
- Designing the type architecture for a new project
- Writing complex generic types
- Making an existing codebase more type-safe
- Creating declaration files for untyped libraries
- Using discriminated unions for state modeling
- Fixing `any` type leaks and type assertion abuse

## Project type structure

```
types/
  index.ts             ← Re-exports everything
  common.ts            ← Shared primitives (Date, ID, Email, etc.)
  api.ts               ← API request/response shapes
  models.ts            ← Domain models (User, Project, Task)
  enums.ts             ← Const enums (if used)
  utils.ts             ← Utility types defined in the project

features/
  auth/
    types.ts           ← Auth-specific types
  projects/
    types.ts           ← Project-specific types

components/
  button/
    button.types.ts    ← Component prop types
```

## Type naming conventions

```ts
// Models: PascalCase singular
type User = { ... };
type Project = { ... };

// Props: PascalCase + "Props" suffix
type ButtonProps = { ... };
type CardProps = { ... };

// State: PascalCase + "State" suffix
type LoadingState = { ... };
type ErrorState = { ... };

// API: PascalCase + "Request" / "Response" suffix
type CreateProjectRequest = { ... };
type CreateProjectResponse = { ... };

// Utility types: PascalCase descriptive
type DeepPartial<T> = { ... };
type Brand<T, B> = { ... };

// Enums: PascalCase
enum ProjectStatus { ... }

// Const objects as enums:
const ProjectStatus = {
  Active: 'active',
  Archived: 'archived',
} as const;
type ProjectStatus = typeof ProjectStatus[keyof typeof ProjectStatus];
```

## Essential patterns

### Discriminated unions for state

```ts
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Usage — TypeScript narrows the type automatically
function handleState(state: RequestState<User>) {
  switch (state.status) {
    case 'idle':
      return <EmptyState />;
    case 'loading':
      return <Spinner />;
    case 'success':
      return <UserCard user={state.data} />;  // state.data is typed as User
    case 'error':
      return <Error message={state.error.message} />;  // state.error is typed as Error
  }
}
```

### Branded types for domain safety

```ts
// Prevent accidentally mixing IDs — UserId ≠ ProjectId even though both are strings
type Brand<T, B extends string> = T & { __brand: B };

type UserId = Brand<string, 'UserId'>;
type ProjectId = Brand<string, 'ProjectId'>;

function getUser(id: UserId): Promise<User> { ... }
function getProject(id: ProjectId): Promise<Project> { ... }

const userId = 'user_123' as UserId;
const projectId = 'proj_456' as ProjectId;

getUser(userId);     // ✓
getUser(projectId);  // ✗ Type error! ProjectId is not assignable to UserId
```

### Exhaustive checking

```ts
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

type Status = 'active' | 'archived' | 'deleted';

function handleStatus(status: Status) {
  switch (status) {
    case 'active': return '🟢';
    case 'archived': return '📦';
    case 'deleted': return '🗑️';
    default: return assertNever(status);
    // If a new status is added but not handled, TypeScript errors at compile time
  }
}
```

### Const assertions and literal types

```ts
// Object as enum
const ROUTES = {
  home: '/',
  projects: '/projects',
  settings: '/settings',
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES];
// Type: '/' | '/projects' | '/settings'

// Array as tuple
const DIRECTIONS = ['north', 'south', 'east', 'west'] as const;
type Direction = typeof DIRECTIONS[number];
// Type: 'north' | 'south' | 'east' | 'west'
```

### Generic constraints

```ts
// Generic with constraint — T must have an 'id' property
function findById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// Generic with keyof constraint — type-safe property access
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map(item => item[key]);
}

// Generic with default
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
  message: string;
}

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;

// Mapped types
type Readonly<T> = { readonly [P in keyof T]: T[P] };

// Template literal types
type EventName = `on${Capitalize<string>}`;
type APIRoute = `/api/${string}`;
```

### Utility type recipes

```ts
// Make specific fields required
type RequireFields<T, K extends keyof T> = T & Required<Pick<T, K>>;

// Make specific fields optional
type PartialFields<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// Deep partial (for recursive objects)
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

// Non-undefined (remove undefined from union)
type NonUndefined<T> = T extends undefined ? never : T;

// ValueOf (get the value type from an object type)
type ValueOf<T> = T[keyof T];

// Function parameters and return types
type Params<T extends (...args: any[]) => any> = Parameters<T>;
type Return<T extends (...args: any[]) => any> = ReturnType<T>;

// Extract specific properties
type PickByValue<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K]
};

// Omit properties by value
type OmitByValue<T, V> = {
  [K in keyof T as T[K] extends V ? never : K]: T[K]
};

// Make all properties writable (undo Readonly)
type Mutable<T> = { -readonly [P in keyof T]: T[P] };
```

### Type guards

```ts
// User-defined type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'email' in value
  );
}

// Assertion function (throws if condition is false)
function assertIsDefined<T>(value: T): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error(`Expected value to be defined, got ${value}`);
  }
}

// Usage
const result: string | undefined = maybeGetString();
assertIsDefined(result);
result.toUpperCase(); // TypeScript knows result is string here

// Type predicate for filtering
function isNonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}

// Filter nulls from arrays type-safely
const items: (string | null)[] = ['a', null, 'b', null, 'c'];
const filtered: string[] = items.filter(isNonNullable);
```

### Component prop types

```ts
// Polymorphic component — renders as different HTML elements
type ButtonProps<C extends React.ElementType = 'button'> = {
  as?: C;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<C>, keyof {
  as?: C; variant?: string; size?: string; children?: React.ReactNode;
}>;

function Button<C extends React.ElementType = 'button'>({
  as,
  variant = 'primary',
  size = 'md',
  children,
  ...rest
}: ButtonProps<C>) {
  const Component = as || 'button';
  return <Component {...rest}>{children}</Component>;
}

// Discriminated union props (mutually exclusive variants)
type CollapsibleProps = {
  children: React.ReactNode;
} & (
  | { open: true; defaultOpen?: never }
  | { open?: never; defaultOpen: boolean }
);
```

### API type patterns

```ts
// Pagination request/response
type PaginatedRequest = {
  page: number;
  pageSize: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
};

type PaginatedResponse<T> = {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    totalItems: number;
    totalPages: number;
  };
};

// API error
type ApiError = {
  code: string;
  message: string;
  details?: Record<string, string[]>;
};

// Typed API client
type ApiClient = {
  get<T>(url: string, config?: RequestConfig): Promise<T>;
  post<T, B>(url: string, body: B, config?: RequestConfig): Promise<T>;
  put<T, B>(url: string, body: B, config?: RequestConfig): Promise<T>;
  delete<T>(url: string, config?: RequestConfig): Promise<T>;
};
```

## Strict tsconfig

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "exactOptionalPropertyTypes": true,
    "forceConsistentCasingInFileNames": true,
    "noUncheckedCallableFunction": true
  }
}
```

`noUncheckedIndexedAccess` is the single most impactful strict flag — it makes array access and object indexing return `T | undefined` instead of `T`, preventing runtime `undefined` errors.

## Type safety checklist

- [ ] `strict: true` in tsconfig
- [ ] `noUncheckedIndexedAccess: true` enabled
- [ ] Zero `any` types (use `unknown` if type is truly unknown)
- [ ] Zero non-null assertions (`!`) — use type guards instead
- [ ] All API boundaries typed (request, response, error)
- [ ] State modeled with discriminated unions
- [ ] Domain IDs use branded types (not raw strings)
- [ ] Exhaustive switch/default checks on all unions
- [ ] Component props use proper generic patterns
- [ ] No `@ts-ignore` without a comment explaining why
- [ ] Environment variables typed via `ProcessEnv` interface

## Anti-patterns I avoid

- `any` — use `unknown` and narrow, or define the type
- `as` type assertions — use type guards or validation instead
- `!` non-null assertion — use null checks or `filter(Boolean)` + type guard
- `$FlowFixMe` / `@ts-ignore` without explanation — fix the type or document why
- Making everything generic — only add generics when you truly need type variance
- `Record<string, any>` — define the shape or use `Record<string, unknown>`
- Overusing enums — prefer `as const` objects with derived types
- Type casting at API boundaries — validate with Zod instead