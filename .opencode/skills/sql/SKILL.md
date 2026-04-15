---
name: sql
description: Design, write, and optimize SQL queries, schemas, and database interactions
---

## What I do

I work with SQL databases — writing queries, designing schemas, and optimizing performance:

- **Schema design** — Normalized tables, appropriate data types, constraints, indexes
- **Query writing** — Idiomatic SQL, CTEs, window functions, proper joins
- **Performance** — Index strategy, query analysis, N+1 prevention, execution plan review
- **Migrations** — Safe schema changes, data migrations, rollback strategies
- **Dialect awareness** — PostgreSQL, MySQL, SQLite, SQL Server specifics

## When to use me

Use this skill when:
- Designing a new database schema or adding tables
- Writing complex queries with joins, aggregations, or window functions
- Optimizing slow queries or adding indexes
- Writing database migrations
- Debugging query logic or data integrity issues

## How I work

1. **Discover the database** — Check which RDBMS is used, find existing schema definitions (migrations, ORM models, `.sql` files), and understand the current table structure.
2. **Understand the data model** — Read entity relationships, constraints, and indexes. Trace foreign keys and understand cardinality.
3. **Write or modify SQL**:
   - Use CTEs for readability, not for performance (some DBs materialize CTEs)
   - Prefer explicit `JOIN` syntax over comma-joins
   - Use parameterized queries — never concatenate user input
   - Choose the right join type (INNER vs LEFT vs CROSS)
4. **Optimize** — Add indexes for common query patterns, use `EXPLAIN`/`EXPLAIN ANALYZE` to verify, avoid `SELECT *`, paginate large results.
5. **Ensure safety** — Transactions for multi-statement changes, backups before destructive operations, test migrations on copies of production data.

## Schema design principles

- Third normal form by default, denormalize intentionally with documentation
- Use appropriate data types (`VARCHAR(n)` not `TEXT` when bounded, `TIMESTAMPTZ` not `TIMESTAMP`)
- Every table has a primary key
- Foreign keys with `ON DELETE` behavior explicitly chosen
- Add `NOT NULL` constraints by default — nullable columns are the exception
- Index foreign keys and columns used in `WHERE`, `JOIN`, `ORDER BY`
- Use database-level constraints (UNIQUE, CHECK) over application-level validation

## Query patterns I follow

- Always alias tables and use the alias for every column
- Use `COALESCE` for null handling, not `IFNULL` or `NVL` (portability)
- Window functions instead of self-joins for running totals, rankings
- `EXISTS`/`NOT EXISTS` instead of `IN`/`NOT IN` with subqueries (handles nulls correctly)
- `UPSERT`/`ON CONFLICT` for idempotent inserts (PostgreSQL), or `MERGE` (SQL Server)
- Parameterized queries always — zero tolerance for SQL injection

## Anti-patterns I avoid

- `SELECT *` in production queries
- N+1 queries (fetch IDs then loop) — use JOINs or `WHERE id IN (...)`
- Cursors when set-based operations work
- Storing computed values without a clear denormalization reason
- Indexing every column "just in case"
- Implicit type conversions in WHERE clauses (kills index usage)