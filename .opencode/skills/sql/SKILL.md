---
name: sql
description: Design, write, and optimize SQL queries, schemas, and database interactions
---

## What I do

I design and optimize SQL:

- **Schema design** — Normalization, relationships, constraints
- **Query writing** — SELECT, INSERT, UPDATE, DELETE, JOINs, CTEs, window functions
- **Optimization** — Indexing, query plans, EXPLAIN ANALYZE
- **Migrations** — Schema changes, data migrations, rollback strategies
- **Anti-patterns** — Common mistakes and how to avoid them

## When to use me

Use this skill when:
- Designing a new database schema
- Writing complex queries or reports
- Optimizing slow queries
- Planning schema migrations
- Choosing between JOIN strategies
- Implementing pagination or search

## Schema design

### Normalization

```sql
-- First Normal Form (1NF): Atomic values
-- Good: Each column contains single value
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL
);

-- Third Normal Form (3NF): No transitive dependencies
-- Good: user_id references users, not duplicated user data
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Junction table for many-to-many
CREATE TABLE order_items (
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

## Query optimization

### Indexing strategies

```sql
-- B-tree index for equality and range queries
CREATE INDEX idx_users_email ON users(email);

-- Composite index for multi-column queries
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Partial index for filtered queries
CREATE INDEX idx_active_products ON products(category_id) WHERE status = 'active';

-- Covering index (includes all queried columns)
CREATE INDEX idx_users_covering ON users(email) INCLUDE (name, created_at);
```

### EXPLAIN ANALYZE

```sql
-- Check query plan
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5;

-- Look for:
-- - Seq Scan vs Index Scan (prefer index)
-- - High execution times
-- - Nested Loop vs Hash Join vs Merge Join
```

## Common patterns

### Pagination

```sql
-- Offset pagination (simple, slow on large offsets)
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 20 OFFSET 1000;

-- Cursor pagination (fast, no skipping)
SELECT * FROM products
WHERE created_at < '2024-01-15 10:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

### CTEs (Common Table Expressions)

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', created_at) as month,
        SUM(total_amount) as revenue,
        COUNT(*) as order_count
    FROM orders
    WHERE created_at >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY DATE_TRUNC('month', created_at)
),
growth AS (
    SELECT
        month,
        revenue,
        order_count,
        LAG(revenue) OVER (ORDER BY month) as prev_revenue,
        (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100 as growth_pct
    FROM monthly_sales
)
SELECT * FROM growth ORDER BY month DESC;
```

### Window functions

```sql
-- Rank products by category
SELECT
    name,
    category_id,
    price,
    RANK() OVER (PARTITION BY category_id ORDER BY price DESC) as price_rank,
    AVG(price) OVER (PARTITION BY category_id) as category_avg_price
FROM products;

-- Running total
SELECT
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;
```

## Anti-patterns

```sql
-- N+1 query problem (fetch users, then fetch orders for each)
-- Fix: Use JOIN
SELECT u.*, o.id as order_id, o.total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- SELECT * in production
-- Fix: Select only needed columns
SELECT id, name, email FROM users;

-- Not using transactions for multi-step operations
-- Fix: Wrap in transaction
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Missing constraints
-- Fix: Add foreign keys, NOT NULL, CHECK constraints
ALTER TABLE orders
ADD CONSTRAINT fk_user
FOREIGN KEY (user_id) REFERENCES users(id);
```

## Migration patterns

```sql
-- Backward-compatible migration: add nullable column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Deploy code that can handle both with and without phone

-- Make column required after all data migrated
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;

-- Index creation (concurrently to avoid locking)
CREATE INDEX CONCURRENTLY idx_users_phone ON users(phone);
```

## Key principles

- Normalize until it hurts, denormalize until it works
- Add indexes for query patterns, not for every column
- Use EXPLAIN ANALYZE to verify optimizations
- Prefer JOINs over multiple queries (N+1 problem)
- Use transactions for multi-step operations
- Add constraints at the database level
- Use parameterized queries (never string interpolation)

## Anti-patterns I avoid

- SELECT * in production queries
- N+1 queries instead of JOINs
- Missing indexes on foreign keys and query columns
- Not using transactions for related changes
- String concatenation for dynamic SQL (SQL injection)
- Not handling NULL values properly
- Over-normalization causing excessive JOINs
- Not analyzing query plans for slow queries