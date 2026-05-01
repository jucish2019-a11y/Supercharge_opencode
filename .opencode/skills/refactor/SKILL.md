---
name: refactor
description: Safe refactoring: identify code smells, plan changes, preserve behavior, and verify
---

## What I do

I refactor code safely by:

- Identifying code smells and improvement opportunities
- Planning changes that preserve existing behavior
- Making incremental, verifiable transformations
- Running tests to confirm no regressions

## When to use me

Use this skill when:
- Code is hard to read or understand
- Duplication exists across modules
- A function or module has too many responsibilities
- A change would be easier after restructuring existing code
- The user asks to "clean up", "simplify", or "reorganize" code

## How I work

1. **Understand current behavior** — Read the code thoroughly. Run existing tests to confirm they pass before starting.
2. **Identify smells** — Look for: long functions, duplicated logic, deep nesting, unclear names, excessive parameters, God objects, feature envy, inappropriate intimacy.
3. **Plan the refactoring** — List each transformation step. Each step should be small enough to verify independently.
4. **Apply one transformation at a time** — Common transformations:
   - Extract function/method
   - Rename for clarity
   - Move code to its proper home
   - Replace conditional with polymorphism
   - Simplify conditional logic
   - Introduce parameter object
5. **Verify after each step** — Run tests after every transformation. If tests fail, revert and reassess.
6. **Clean up** — Remove dead code, unused imports, unreachable branches after transformations are complete.

## Code smell catalog

### Long function
```ts
// BEFORE: Function doing too much
function processOrder(order: Order) {
  // Validate order
  if (!order.items.length) throw new Error('Empty order');
  if (order.total <= 0) throw new Error('Invalid total');

  // Calculate totals
  let subtotal = 0;
  for (const item of order.items) {
    subtotal += item.price * item.quantity;
  }
  const tax = subtotal * 0.08;
  const total = subtotal + tax;

  // Save to database
  const saved = db.order.create({
    data: { ...order, subtotal, tax, total },
  });

  // Send confirmation email
  sendEmail(order.customerEmail, 'order-confirmation', { orderId: saved.id });

  // Update inventory
  for (const item of order.items) {
    db.product.update({
      where: { id: item.productId },
      data: { stock: { decrement: item.quantity } },
    });
  }

  return saved;
}

// AFTER: Extracted responsibilities
function validateOrder(order: Order): void {
  if (!order.items.length) throw new Error('Empty order');
  if (order.total <= 0) throw new Error('Invalid total');
}

function calculateTotals(items: OrderItem[]): { subtotal: number; tax: number; total: number } {
  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const tax = subtotal * 0.08;
  return { subtotal, tax, total: subtotal + tax };
}

async function saveOrder(order: Order, totals: Totals): Promise<Order> {
  return db.order.create({
    data: { ...order, ...totals },
  });
}

async function processOrder(order: Order): Promise<Order> {
  validateOrder(order);
  const totals = calculateTotals(order.items);
  const saved = await saveOrder(order, totals);
  await sendOrderConfirmation(saved);
  await updateInventory(order.items);
  return saved;
}
```

### Duplicated code
```ts
// BEFORE: Duplicated validation
function createUser(data: UserData) {
  if (!data.email?.includes('@')) throw new Error('Invalid email');
  if (data.name?.length < 2) throw new Error('Name too short');
  // ... create logic
}

function updateUser(id: string, data: UserData) {
  if (!data.email?.includes('@')) throw new Error('Invalid email');
  if (data.name?.length < 2) throw new Error('Name too short');
  // ... update logic
}

// AFTER: Extracted validation
const UserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

function validateUser(data: unknown): UserData {
  return UserSchema.parse(data);
}

function createUser(data: unknown) {
  const valid = validateUser(data);
  // ... create logic
}

function updateUser(id: string, data: unknown) {
  const valid = validateUser(data);
  // ... update logic
}
```

### Deep nesting
```ts
// BEFORE: Pyramid of doom
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (response.ok) {
      const data = await response.json();
      if (data.items) {
        for (const item of data.items) {
          if (item.active) {
            await process(item);
          }
        }
      }
    }
  } catch (error) {
    console.error(error);
  }
}

// AFTER: Guard clauses and early returns
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) return;

    const data = await response.json();
    if (!data.items) return;

    const activeItems = data.items.filter(item => item.active);
    await Promise.all(activeItems.map(process));
  } catch (error) {
    console.error(error);
  }
}
```

### Magic numbers/strings
```ts
// BEFORE: Magic values
if (user.role === 2) { /* ... */ }
if (status === 'active' && count > 5) { /* ... */ }

// AFTER: Named constants
const ROLES = {
  USER: 1,
  ADMIN: 2,
  MODERATOR: 3,
} as const;

const MAX_RETRY_COUNT = 5;
const STATUS_ACTIVE = 'active';

if (user.role === ROLES.ADMIN) { /* ... */ }
if (status === STATUS_ACTIVE && count > MAX_RETRY_COUNT) { /* ... */ }
```

## Refactoring recipes

### Extract method
```ts
// BEFORE
function processPayment(order: Order) {
  // 20 lines of validation
  // 15 lines of calculation
  // 10 lines of database update
  // 5 lines of email notification
}

// AFTER
function processPayment(order: Order) {
  validatePayment(order);
  const totals = calculatePaymentTotals(order);
  const payment = savePayment(order, totals);
  notifyPaymentComplete(payment);
}
```

### Replace conditional with polymorphism
```ts
// BEFORE
function calculateShipping(order: Order): number {
  if (order.type === 'standard') return 5;
  if (order.type === 'express') return 15;
  if (order.type === 'free') return 0;
  return 5;
}

// AFTER
interface ShippingStrategy {
  calculate(order: Order): number;
}

class StandardShipping implements ShippingStrategy {
  calculate() { return 5; }
}

class ExpressShipping implements ShippingStrategy {
  calculate() { return 15; }
}

class FreeShipping implements ShippingStrategy {
  calculate() { return 0; }
}

const strategies: Record<string, ShippingStrategy> = {
  standard: new StandardShipping(),
  express: new ExpressShipping(),
  free: new FreeShipping(),
};

function calculateShipping(order: Order): number {
  return strategies[order.type]?.calculate(order) ?? 5;
}
```

### Introduce parameter object
```ts
// BEFORE
function createUser(
  firstName: string,
  lastName: string,
  email: string,
  phone: string,
  address: string,
  city: string,
  country: string
) { /* ... */ }

// AFTER
interface UserProfile {
  firstName: string;
  lastName: string;
  email: string;
  phone: string;
  address: Address;
}

interface Address {
  street: string;
  city: string;
  country: string;
}

function createUser(profile: UserProfile) { /* ... */ }
```

## Verification strategies

```bash
# 1. Run tests before refactoring
npm test

# 2. Make one small change
# 3. Run tests again
npm test

# 4. If tests fail, revert and try again
git checkout -- src/file.ts

# 5. Commit frequently
git add src/file.ts
git commit -m "refactor: extract validation logic"

# 6. Run full test suite after all changes
npm test -- --coverage

# 7. Check for type errors
npm run typecheck
```

## Key principles

- **Behavior preservation**: Refactoring must not change observable behavior
- **Small steps**: Each transformation is independently verifiable
- **Test first**: If tests don't exist, write them before refactoring
- **No mixed changes**: Don't refactor and add features in the same change
- **Revert on failure**: If any step breaks tests, revert it rather than patching

## Anti-patterns I avoid

- Refactoring without tests — dangerous and error-prone
- Mixing refactoring with feature changes in the same commit
- Large "big bang" refactorings that touch many files
- Refactoring just for the sake of it — only when it improves maintainability
- Not running tests after each small change
- Changing behavior "accidentally" while refactoring
- Not documenting why a refactoring was done in the commit message
- Ignoring type errors introduced by refactoring
- Refactoring code that's about to be deleted