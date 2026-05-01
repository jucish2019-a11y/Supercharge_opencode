---
name: data-validation
description: Validate and sanitize data — Zod schemas, form validation, API request validation, custom validators, sanitization, and business rule enforcement
---

## What I do

I implement data validation at every boundary of an application:

- **Schema validation** — Zod, Yup, Joi for structured data validation
- **API validation** — Request body, query params, path params validation
- **Form validation** — Client-side and server-side form validation
- **Sanitization** — XSS prevention, SQL injection prevention, input cleaning
- **Business rules** — Domain-specific validation beyond schema
- **Error formatting** — User-friendly validation error messages

## When to use me

Use this skill when:
- Validating API request payloads
- Building forms with client and server validation
- Sanitizing user inputs to prevent injection attacks
- Implementing complex business rule validation
- Creating reusable validation schemas
- Converting between external and internal data formats

## Validation at boundaries

```
Data enters the application
├── API requests (HTTP)
│   └── Validate body, query, params, headers
├── Forms (user input)
│   └── Validate on client (UX) + server (security)
├── File uploads
│   └── Validate type, size, content
├── Webhooks / external APIs
│   └── Validate payload shape and signature
├── Database reads
│   └── Validate with Prisma Zod or similar
└── CLI arguments / env vars
    └── Validate at startup
```

## Zod schemas

### Basic primitives

```ts
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email('Invalid email address'),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(['user', 'admin', 'moderator']),
  isActive: z.boolean().default(true),
  createdAt: z.coerce.date(),
  metadata: z.record(z.unknown()).optional(),
});

type User = z.infer<typeof UserSchema>;
```

### Advanced patterns

```ts
// Custom validation
const PasswordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Password must contain an uppercase letter')
  .regex(/[a-z]/, 'Password must contain a lowercase letter')
  .regex(/[0-9]/, 'Password must contain a number')
  .regex(/[^A-Za-z0-9]/, 'Password must contain a special character');

// Transform and refine
const SlugSchema = z.string()
  .min(1)
  .max(100)
  .regex(/^[a-z0-9-]+$/, 'Slug must be lowercase alphanumeric with hyphens')
  .transform(val => val.toLowerCase().replace(/-+/g, '-'));

// Union types
const SearchQuerySchema = z.union([
  z.object({ type: z.literal('user'), userId: z.string().uuid() }),
  z.object({ type: z.literal('team'), teamId: z.string().uuid() }),
  z.object({ type: z.literal('global'), query: z.string().min(1) }),
]);

// Recursive types
const CategorySchema: z.ZodType<Category> = z.lazy(() =>
  z.object({
    id: z.string(),
    name: z.string(),
    children: z.array(CategorySchema).optional(),
  })
);

// Preprocessing
const NumberFromStringSchema = z.preprocess(
  (val) => (typeof val === 'string' ? parseFloat(val) : val),
  z.number()
);
```

## API request validation

### Middleware pattern

```ts
import { z } from 'zod';
import { NextRequest, NextResponse } from 'next/server';

function validateBody<T>(schema: z.ZodSchema<T>) {
  return async (req: NextRequest): Promise<T> => {
    const body = await req.json();
    const result = schema.safeParse(body);

    if (!result.success) {
      const errors = result.error.errors.map(e => ({
        path: e.path.join('.'),
        message: e.message,
      }));

      throw new ValidationError('Invalid request body', errors);
    }

    return result.data;
  };
}

function validateQuery<T>(schema: z.ZodSchema<T>) {
  return (req: NextRequest): T => {
    const searchParams = Object.fromEntries(req.nextUrl.searchParams);
    const result = schema.safeParse(searchParams);

    if (!result.success) {
      throw new ValidationError('Invalid query parameters', result.error.errors);
    }

    return result.data;
  };
}

// Usage
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  password: z.string().min(8),
});

export async function POST(req: NextRequest) {
  try {
    const body = await validateBody(CreateUserSchema)(req);

    const user = await db.user.create({
      data: {
        email: body.email,
        name: body.name,
        password: await hashPassword(body.password),
      },
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof ValidationError) {
      return NextResponse.json({ errors: error.errors }, { status: 422 });
    }
    throw error;
  }
}
```

## Form validation

### React Hook Form + Zod

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const FormSchema = z.object({
  email: z.string().email('Please enter a valid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
  terms: z.literal(true, {
    errorMap: () => ({ message: 'You must accept the terms' }),
  }),
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

type FormData = z.infer<typeof FormSchema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(FormSchema),
  });

  const onSubmit = async (data: FormData) => {
    await fetch('/api/auth/signup', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} placeholder="Email" />
      {errors.email && <span>{errors.email.message}</span>}

      <input type="password" {...register('password')} placeholder="Password" />
      {errors.password && <span>{errors.password.message}</span>}

      <input type="password" {...register('confirmPassword')} placeholder="Confirm Password" />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <label>
        <input type="checkbox" {...register('terms')} />
        I agree to the terms
      </label>
      {errors.terms && <span>{errors.terms.message}</span>}

      <button type="submit" disabled={isSubmitting}>Sign Up</button>
    </form>
  );
}
```

## Sanitization

### XSS prevention

```ts
import DOMPurify from 'isomorphic-dompurify';

function sanitizeHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'target'],
  });
}

// For user-generated content displayed as HTML
function renderUserContent(content: string) {
  const clean = sanitizeHtml(content);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}

// Better: never render HTML from users
function renderUserContentSafe(content: string) {
  return <div className="whitespace-pre-wrap">{content}</div>;
}
```

### SQL injection prevention

```ts
// NEVER concatenate user input into SQL
// BAD:
const query = `SELECT * FROM users WHERE email = '${userInput}'`;

// GOOD (parameterized queries):
const users = await db.$queryRaw`
  SELECT * FROM users WHERE email = ${userInput}
`;

// GOOD (ORM):
const user = await db.user.findUnique({
  where: { email: userInput },
});
```

## Business rule validation

```ts
class OrderValidator {
  async validate(order: OrderData): Promise<ValidationResult> {
    const errors: ValidationError[] = [];

    // Schema validation
    const schemaResult = OrderSchema.safeParse(order);
    if (!schemaResult.success) {
      errors.push(...schemaResult.error.errors);
    }

    // Business rules
    const product = await db.product.findUnique({
      where: { id: order.productId },
    });

    if (!product) {
      errors.push({ path: 'productId', message: 'Product not found' });
    } else if (!product.inStock) {
      errors.push({ path: 'productId', message: 'Product is out of stock' });
    } else if (product.stockQuantity < order.quantity) {
      errors.push({
        path: 'quantity',
        message: `Only ${product.stockQuantity} items available`,
      });
    }

    // User-specific rules
    const user = await db.user.findUnique({ where: { id: order.userId } });
    if (user?.suspended) {
      errors.push({ path: 'userId', message: 'User account is suspended' });
    }

    // Time-based rules
    const now = new Date();
    if (order.deliveryDate && order.deliveryDate < now) {
      errors.push({ path: 'deliveryDate', message: 'Delivery date must be in the future' });
    }

    return {
      valid: errors.length === 0,
      errors,
    };
  }
}
```

## Error formatting

```ts
function formatZodErrors(error: z.ZodError): Record<string, string[]> {
  const formatted: Record<string, string[]> = {};

  for (const err of error.errors) {
    const path = err.path.join('.');
    if (!formatted[path]) formatted[path] = [];
    formatted[path].push(err.message);
  }

  return formatted;
}

function formatErrorsForApi(errors: ValidationError[]) {
  return {
    message: 'Validation failed',
    errors: errors.map(e => ({
      field: e.path,
      message: e.message,
    })),
  };
}

function formatErrorsForForm(errors: ValidationError[]) {
  const formatted: Record<string, string> = {};
  for (const error of errors) {
    formatted[error.path] = error.message;
  }
  return formatted;
}
```

## Quality checklist

- [ ] All API inputs validated with schemas before processing
- [ ] Form validation on both client (UX) and server (security)
- [ ] User inputs sanitized before storage or display
- [ ] SQL queries use parameterized statements — never concatenation
- [ ] HTML from users sanitized with DOMPurify or rejected
- [ ] Business rules validated separately from schema validation
- [ ] Error messages are user-friendly and specific
- [ ] Validation schemas are reusable and composable
- [ ] File uploads validated for type, size, and content
- [ ] Rate limiting applied to validation-heavy endpoints

## Anti-patterns I avoid

- Trusting client-side validation alone — always validate server-side
- Using `as` type assertions instead of runtime validation
- Concatenating user input into SQL queries
- Displaying raw user HTML without sanitization
- Generic "validation failed" messages without field-specific details
- Validating in business logic instead of at the boundary
- Duplicating validation logic between client and server
- Not validating query parameters and headers
- Using regex for complex validation (email, URL) — use libraries