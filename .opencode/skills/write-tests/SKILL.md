---
name: write-tests
description: Generate thorough tests following project conventions and best practices
---

## What I do

I write comprehensive tests for code:

- Discover the project's test framework and conventions
- Cover happy paths, edge cases, and error paths
- Follow existing test structure and naming patterns
- Ensure tests are deterministic and isolated

## When to use me

Use this skill when:
- Writing tests for new or existing code
- Increasing test coverage for a module
- Adding regression tests after fixing a bug
- Setting up test infrastructure for a new project

## How I work

1. **Discover conventions** — Find the test framework, runner config, assertion style, directory structure, and naming patterns by examining existing tests.
2. **Identify what to test** — For each function/method: happy path inputs, boundary values, invalid inputs, error conditions, side effects.
3. **Follow existing patterns** — Match import style, describe/it structure, setup/teardown, mocking approach, and assertion style of existing tests.
4. **Write isolated tests** — Each test should be independent. No shared mutable state between tests. Mock external dependencies (network, filesystem, time) when needed.
5. **Make tests deterministic** — Use fixed values, not random or time-dependent ones. Control async ordering where relevant.
6. **Run and verify** — Execute the tests and confirm they pass. Fix any failures.

## Test structure per case

- **Unit tests**: Test one function in isolation. Mock dependencies.
- **Integration tests**: Test multiple modules together. Use real dependencies when possible.
- **Regression tests**: Write a test that reproduces the bug first (should fail), then verify it passes after the fix.

## Jest/Vitest patterns

### Basic test structure

```ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { calculateTotal } from './cart';

describe('calculateTotal', () => {
  it('should calculate total for single item', () => {
    const items = [{ price: 10, quantity: 1 }];
    expect(calculateTotal(items)).toBe(10);
  });

  it('should calculate total for multiple items', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 3 },
    ];
    expect(calculateTotal(items)).toBe(35);
  });

  it('should return 0 for empty cart', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('should apply discount', () => {
    const items = [{ price: 100, quantity: 1 }];
    expect(calculateTotal(items, { discount: 0.1 })).toBe(90);
  });
});
```

### Mocking

```ts
import { vi } from 'vitest';

// Mock a module
vi.mock('./api', () => ({
  fetchUser: vi.fn(),
}));

// Mock with implementation
import { fetchUser } from './api';

beforeEach(() => {
  vi.clearAllMocks();
});

it('should handle user fetch', async () => {
  vi.mocked(fetchUser).mockResolvedValue({ id: '1', name: 'John' });

  const result = await loadUser('1');
  expect(result).toEqual({ id: '1', name: 'John' });
  expect(fetchUser).toHaveBeenCalledWith('1');
});

// Mock timers
vi.useFakeTimers();

it('should debounce search', () => {
  const onSearch = vi.fn();
  const debounced = debounce(onSearch, 300);

  debounced('test');
  expect(onSearch).not.toHaveBeenCalled();

  vi.advanceTimersByTime(300);
  expect(onSearch).toHaveBeenCalledWith('test');
});
```

### Testing React components

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

describe('Counter', () => {
  it('should render initial count', () => {
    render(<Counter initial={5} />);
    expect(screen.getByText('5')).toBeInTheDocument();
  });

  it('should increment on button click', async () => {
    const user = userEvent.setup();
    render(<Counter initial={0} />);

    await user.click(screen.getByRole('button', { name: /increment/i }));
    expect(screen.getByText('1')).toBeInTheDocument();
  });

  it('should not decrement below zero', async () => {
    const user = userEvent.setup();
    render(<Counter initial={0} />);

    await user.click(screen.getByRole('button', { name: /decrement/i }));
    expect(screen.getByText('0')).toBeInTheDocument();
  });

  it('should call onChange callback', async () => {
    const user = userEvent.setup();
    const onChange = vi.fn();
    render(<Counter initial={0} onChange={onChange} />);

    await user.click(screen.getByRole('button', { name: /increment/i }));
    expect(onChange).toHaveBeenCalledWith(1);
  });
});
```

### Testing async code

```ts
describe('async operations', () => {
  it('should handle loading state', async () => {
    render(<UserList />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
    });
  });

  it('should handle error state', async () => {
    server.use(
      http.get('/api/users', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );

    render(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('Error loading users')).toBeInTheDocument();
    });
  });
});
```

### Testing hooks

```ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('should initialize with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('should increment', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});
```

## Playwright E2E tests

```ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('should login with valid credentials', async ({ page }) => {
    await page.fill('[name="email"]', 'user@example.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toContainText('Dashboard');
  });

  test('should show error with invalid credentials', async ({ page }) => {
    await page.fill('[name="email"]', 'wrong@example.com');
    await page.fill('[name="password"]', 'wrong');
    await page.click('button[type="submit"]');

    await expect(page.locator('[role="alert"]')).toContainText('Invalid credentials');
  });
});
```

## Test data factories

```ts
// factories/user.ts
import { faker } from '@faker-js/faker';

export function createUser(overrides: Partial<User> = {}): User {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    name: faker.person.fullName(),
    role: 'user',
    createdAt: faker.date.past(),
    ...overrides,
  };
}

export function createUsers(count: number): User[] {
  return Array.from({ length: count }, () => createUser());
}

// Usage in tests
const admin = createUser({ role: 'admin' });
const users = createUsers(10);
```

## Guidelines

- Always check for existing tests before writing new ones
- Name tests descriptively: "should [expected behavior] when [condition]"
- Test error paths, not just happy paths
- Avoid testing implementation details — test behavior
- Never skip tests with TODO — write the test or don't create it
- Run lint and typecheck after writing tests

## Quality checklist

- [ ] Happy path tested for each function
- [ ] Edge cases covered (empty input, null, maximum values)
- [ ] Error paths tested
- [ ] Async operations tested (loading, success, error states)
- [ ] Tests are deterministic (no random data, no time dependencies)
- [ ] Tests are isolated (no shared state between tests)
- [ ] External dependencies mocked
- [ ] Test names describe behavior, not implementation
- [ ] Coverage report reviewed for gaps
- [ ] No test.only or test.skip left in code

## Anti-patterns I avoid

- Testing implementation details instead of behavior
- Using real API calls in unit tests — always mock
- Shared mutable state between tests
- Time-dependent tests (use fake timers)
- Random data in tests (use deterministic factories)
- Not cleaning up after tests (clear mocks, reset state)
- Writing tests after the fact without understanding requirements
- Testing private functions directly
- Over-mocking (mocking everything including the unit under test)
- Ignoring failing tests or commenting them out