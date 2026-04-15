---
name: testing-strategy
description: Design testing strategies — test pyramid, coverage targets, what to test at each level, E2E vs integration vs unit, and when to write which test
---

## What I do

I design testing strategies that give confidence without slowing development:

- **Test pyramid** — Unit → Integration → E2E — how many of each, what goes where
- **What to test** — Business logic, edge cases, API contracts, user flows
- **What NOT to test** — Implementation details, third-party code, trivial code
- **Test architecture** — Fixtures, factories, test database, mocking strategy
- **Coverage targets** — Meaningful metrics, not arbitrary percentages
- **CI integration** — Fast feedback, parallel execution, flaky test management

## When to use me

Use this skill when:
- Setting up testing for a new project
- Deciding what kind of test to write for a specific case
- Choosing testing frameworks and tools
- Designing test architecture (fixtures, factories, database)
- Setting coverage targets and CI test gates
- Dealing with flaky or slow test suites

## The test pyramid

```
        ╱╲
       ╱  ╲         E2E (5-10%)
      ╱    ╲        Few, slow, expensive, high confidence
     ╱──────╲
    ╱        ╲      Integration (20-30%)
   ╱          ╲     API contracts, component interaction, DB queries
  ╱────────────╲
 ╱              ╲    Unit (60-75%)
╱                ╲   Pure logic, edge cases, transforms, validation
──────────────────
```

| Level | Speed | Count | What it tests | Tools |
|-------|-------|-------|---------------|-------|
| **Unit** | <10ms | Many | Pure functions, business logic, validation | Vitest, Jest |
| **Integration** | 100ms-1s | Moderate | API routes with DB, component + store, hooks + API | Vitest, Testing Library, MSW |
| **E2E** | 5-30s | Few | Critical user flows across the full stack | Playwright, Cypress |

### What goes at each level

**Unit tests** (write many, run fast):
- Pure business logic (calculations, transforms, validators)
- Zod schema validation (does the schema accept/reject correctly?)
- Utility functions (formatDate, parseCurrency, buildQueryString)
- Reducers and state machines
- Error classes and error formatting
- Permission checks (can user X do Y?)
- Custom React hooks (via renderHook)

**Integration tests** (write moderate, test boundaries):
- API route handlers with real database (test DB)
- React components that fetch data (with MSW)
- Database queries and ORM configurations
- Auth middleware + session handling
- Form submission flow (component + validation + API)
- Email/rendering services

**E2E tests** (write few, test critical paths):
- Signup → create project → add task → complete task
- Login → view dashboard → navigate to settings
- Checkout flow: cart → payment → confirmation
- Error recovery: API down → retry → success

## What NOT to test

```
Don't test:
  ✗ Third-party library behavior (Zod, React, Prisma test their own code)
  ✗ Implementation details (which method is called, internal state)
  ✗ Simple getter/setter with no logic
  ✗ CSS properties (unless visual regression)
  ✗ Constants and type definitions
  ✗ Framework boilerplate (Next.js routing, Express setup)
  
Test instead:
  ✓ How YOU use the library (your schemas, your configs)
  ✓ The contract: given input, does output match expectation?
  ✓ The user behavior, not the code structure
  ✓ Visual output via visual regression (if needed)
  ✓ That types are correct (via TypeScript compiler)
  ✓ That your routes return correct data
```

## Testing React components

### Test behavior, not implementation

```tsx
// ✗ DON'T test implementation details
test('calls handleSubmit on click', () => {
  const onSubmit = jest.fn();
  render(<Form onSubmit={onSubmit} />);
  fireEvent.click(screen.getByRole('button'));
  expect(onSubmit).toHaveBeenCalled(); // Tests internal wiring
});

// ✓ DO test user behavior
test('submits the form with valid data', async () => {
  const user = userEvent.setup();
  render(<Form />);

  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password123');
  await user.click(screen.getByRole('button', { name: 'Sign in' }));

  expect(await screen.findByText('Welcome back')).toBeInTheDocument();
});
```

### Component testing patterns

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Test: component renders correctly
test('renders project name', () => {
  render(<ProjectCard project={mockProject} />);
  expect(screen.getByText('Project Alpha')).toBeInTheDocument();
});

// Test: interaction works
test('clicking archive shows confirmation', async () => {
  const user = userEvent.setup();
  render(<ProjectCard project={mockProject} />);
  await user.click(screen.getByRole('button', { name: 'Archive' }));
  expect(screen.getByText('Archive this project?')).toBeInTheDocument();
});

// Test: loading state
test('shows skeleton while loading', () => {
  render(<ProjectCard loading />);
  expect(screen.getByTestId('skeleton')).toBeInTheDocument();
});

// Test: error state
test('shows error message on failure', () => {
  render(<ProjectCard error={new Error('Failed to load')} />);
  expect(screen.getByText('Failed to load')).toBeInTheDocument();
});

// Test: empty state
test('shows empty state when no projects', () => {
  render(<ProjectList projects={[]} />);
  expect(screen.getByText('No projects yet')).toBeInTheDocument();
});
```

## API testing patterns

### Testing route handlers with database

```ts
import { build } from '@/test/helpers';
import { db } from '@/lib/db';

beforeEach(async () => {
  await db.$executeRaw`TRUNCATE TABLE projects CASCADE`;
});

test('POST /api/projects creates a project', async () => {
  const app = build();

  const res = await app.request('/api/projects', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
    body: JSON.stringify({ name: 'New Project' }),
  });

  expect(res.status).toBe(201);
  const body = await res.json();
  expect(body.data.name).toBe('New Project');
});

test('POST /api/projects validates input', async () => {
  const app = build();

  const res = await app.request('/api/projects', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
    body: JSON.stringify({ name: '' }),
  });

  expect(res.status).toBe(400);
  const body = await res.json();
  expect(body.error.code).toBe('VALIDATION_ERROR');
});
```

### Mock Service Worker (MSW) for client-side API mocking

```ts
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer(
  http.get('/api/projects', () => {
    return HttpResponse.json({ data: [mockProject] });
  }),
  http.post('/api/projects', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ data: { id: '1', ...body } }, { status: 201 });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Test factories and fixtures

```ts
// test/factory.ts
import { faker } from '@faker-js/faker';

export function buildProject(overrides: Partial<Project> = {}): Project {
  return {
    id: faker.string.uuid(),
    name: faker.company.name(),
    description: faker.lorem.sentence(),
    status: 'active',
    createdAt: faker.date.recent(),
    updatedAt: faker.date.recent(),
    ...overrides,
  };
}

export function buildUser(overrides: Partial<User> = {}): User {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    name: faker.person.fullName(),
    role: 'member',
    createdAt: faker.date.recent(),
    ...overrides,
  };
}

// Usage
const project = buildProject({ status: 'archived' });
const admin = buildUser({ role: 'admin' });
```

## E2E testing with Playwright

```ts
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test('user can sign up and create a project', async ({ page }) => {
  await page.goto('/register');

  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'SecurePass123!');
  await page.fill('[name="name"]', 'Test User');
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL('/onboarding');

  await page.click('text=Create your first project');
  await page.fill('[name="name"]', 'My First Project');
  await page.click('text=Create project');

  await expect(page.locator('h1')).toContainText('My First Project');
});

test('shows error on invalid login', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'wrong@example.com');
  await page.fill('[name="password"]', 'wrongpassword');
  await page.click('button[type="submit"]');

  await expect(page.locator('[role="alert"]')).toContainText('Invalid credentials');
});
```

## Coverage strategy

### Target coverage

| Level | Target | Rationale |
|-------|--------|-----------|
| **Business logic** | 90%+ | Core value — must be tested |
| **API routes** | 80%+ | Integration tests with DB |
| **Components** | 60-70% | Key interactions, not every variant |
| **Utilities** | 90%+ | Pure functions, easy to test |
| **Overall** | 70-80% | Beyond this, diminishing returns |

### What 100% coverage means

- 100% line coverage ≠ 100% bug-free
- Coverage measures which lines executed, not which scenarios tested
- Better: 80% coverage with meaningful assertions > 100% coverage with weak assertions
- Never merge-require coverage — it encourages low-quality tests
- Use coverage to find UNTESTED code, not to enforce a percentage

## Flaky test management

```
Prevention:
  - Use deterministic data (factories, not random API data)
  - Use test database, reset between tests
  - Use MSW for API mocking (no real network calls)
  - Mock time (use fake timers)
  - Don't test animations or transitions (non-deterministic)
  - Use waitFor/findBy for async (not setTimeout)
  
Detection:
  - Run E2E tests 3x in CI — if any fail, it's flaky
  - Quarantine flaky tests (run but don't block CI)
  - Track flake rate per test
  
Recovery:
  - Retry flaky E2E tests 2x before failing
  - Log screenshots and DOM on failure for debugging
```

## CI test configuration

```
Pipeline:
  1. Lint + Type check (10s) — fail fast
  2. Unit tests (30s) — parallel, no DB needed
  3. Integration tests (2min) — test DB, parallel
  4. Build (1min) — ensure it compiles
  5. E2E tests (5min) — separate job, real browser

Rules:
  - Unit tests run on every PR
  - Integration tests run on every PR
  - E2E tests run on merge to main + nightly
  - Critical path E2E runs on every PR (subset)
```

## Quality checklist

- [ ] Test pyramid follows 60-70% unit / 20-30% integration / 5-10% E2E
- [ ] Business logic has >90% coverage
- [ ] All error paths have tests (not just happy path)
- [ ] Test factories generate realistic data
- [ ] Integration tests use real test database
- [ ] API mocking with MSW for client-side tests
- [ ] E2E tests cover critical user flows only
- [ ] Flaky tests quarantined, not ignored
- [ ] CI pipeline runs lint → unit → integration → build → E2E
- [ ] Coverage reported but not enforced as a gate
- [ ] No testing of implementation details
- [ ] Tests are independent (no order dependencies)

## Anti-patterns I avoid

- Testing implementation details (which function is called, internal state)
- Snapshot testing for anything except serialized output
- 100% coverage as a merge requirement — encourages bad tests
- Mocking everything — mock external APIs, not internal modules
- Sharing state between tests (test order dependency)
- UI component tests that verify CSS classes — test behavior, not styling
- Running E2E tests on every PR for all flows — too slow, run critical paths only
- Using setTimeout in tests — use findBy/waitFor for async