---
name: database-migration
description: Safe schema changes and data migrations with rollback strategies and zero-downtime deployment
---

## What I do

I handle database schema and data migrations safely — from planning the change to verifying it in production:

- Design reversible schema changes with rollback paths
- Write data migration scripts for transforming existing data
- Ensure zero-downtime deployment compatibility via expand-then-contract
- Handle large table migrations without locking
- Plan rollback strategies for every change

## When to use me

Use this skill when:
- Adding, removing, or altering tables and columns
- Changing data types or constraints
- Migrating data between schemas or renaming columns
- Adding indexes to large tables without locking
- Fixing schema drift between environments
- Planning a deployment that includes database changes

## How I work

### Checker mode (assessing migration risk)

1. **Read the current schema** — Inspect migration files, model definitions, and schema docs. Understand the current state.
2. **Identify the change type** — Classify as: additive (low risk), destructive (high risk), or transformative (medium risk).
3. **Assess table size** — How many rows? Is this a large table that needs concurrent operations?
4. **Check application coupling** — Does the application reference the changing columns/tables directly? In queries? In ORM models?
5. **Estimate downtime risk** — Can this be done online, or does it require a maintenance window?

### Applier mode (performing the migration)

1. **Analyze the current schema** — Read migration files, model definitions, and schema docs. Understand the current state.
2. **Plan the migration** — Determine whether the change requires one step or multiple. Prefer expand-then-contract for production safety.
3. **Write the migration** — Use the project's migration tool (Alembic, Django, Prisma, Knex, Flyway, etc.). Follow existing naming conventions.
4. **Include a rollback** — Every migration must have a down/revert path. Test it.
5. **Handle data migrations separately** — Don't mix schema changes and data transforms in one migration. Create separate migrations for data transforms.
6. **Consider production impact** — For large tables: add columns as nullable first, backfill in batches, then add not-null constraints. Create indexes concurrently where supported.
7. **Verify** — Run migrations up and down. Check that the application still works with both old and new schemas during transition.

## Migration types and strategies

### Additive changes (low risk)

Adding a new table, column, or index that doesn't affect existing code:

```
-- Safe: application ignores the new column entirely
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- Safe: new table, no existing code references it
CREATE TABLE notifications (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  message TEXT NOT NULL,
  read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Safe: new index on existing column
CREATE INDEX CONCURRENTLY ON users (email);  -- PostgreSQL
```

Why it's safe: The application doesn't know about the new structure, so it continues working. Rollback is `DROP COLUMN` or `DROP TABLE`.

### Destructive changes (high risk)

Removing a column, table, or constraint that existing code references:

```
-- DANGEROUS: application code references this column
ALTER TABLE users DROP COLUMN legacy_field;

-- DANGEROUS: application code queries this table
DROP TABLE legacy_reports;
```

Strategy: Use expand-then-contract (see below).

### Transformative changes (medium risk)

Changing a column type, renaming a column, altering a constraint:

```
-- MEDIUM RISK: data conversion may lose precision
ALTER TABLE products ALTER COLUMN price TYPE DECIMAL(10,2);

-- MEDIUM RISK: renaming breaks all queries that reference the old name
ALTER TABLE users RENAME COLUMN name TO full_name;
```

Strategy: Expand-then-contract for renames. For type changes, add a new column, backfill, then swap.

## Expand-then-contract pattern

For any change that could break a running application, use the three-step pattern:

### Step 1: Expand (add the new structure)

```
-- Add new column alongside old one
ALTER TABLE users ADD COLUMN full_name TEXT;

-- Application: dual-write to both columns
-- UPDATE: set full_name = name WHERE full_name IS NULL (backfill)
```

Deploy this. Application now reads from old column and writes to both. Rollback: drop the new column.

### Step 2: Migrate (backfill data)

```
-- Backfill existing rows
UPDATE users SET full_name = name WHERE full_name IS NULL;

-- For large tables, batch the update:
-- Process 10,000 rows at a time with a sleep between batches
-- to avoid locking the table
```

Deploy this as a separate migration. No application changes needed. Rollback: no-op (data is already there).

### Step 3: Contract (remove the old structure)

```
-- Switch application to read from full_name instead of name
-- Then remove the old column in a later deployment

ALTER TABLE users DROP COLUMN name;
```

Deploy this only after the application has been updated to use the new column. Rollback: re-add the old column (but you've lost the data — so keep a backup).

## Large table migrations

Tables with millions of rows need special handling to avoid locks:

### Adding indexes

```
-- PostgreSQL: CONCURRENTLY doesn't lock writes
CREATE INDEX CONCURRENTLY ON orders (user_id);

-- MySQL: Use ALGORITHM=INPLACE, LOCK=NONE
ALTER TABLE orders ADD INDEX idx_user_id (user_id), ALGORITHM=INPLACE, LOCK=NONE;
```

### Adding columns with defaults

```
-- DANGEROUS on large tables (rewrites every row):
ALTER TABLE orders ADD COLUMN status TEXT DEFAULT 'pending';

-- SAFE: Add nullable, then set default, then backfill
ALTER TABLE orders ADD COLUMN status TEXT;
ALTER TABLE orders ALTER COLUMN status SET DEFAULT 'pending';

-- Then backfill in batches (see below)
```

### Batch updates for backfilling

```sql
-- Backfill in batches of 10,000 rows
-- Run this repeatedly until 0 rows are updated
UPDATE orders
SET status = 'pending'
WHERE status IS NULL
LIMIT 10000;

-- Or use a cursor-based approach for very large tables:
UPDATE orders
SET status = 'pending'
WHERE id IN (
  SELECT id FROM orders WHERE status IS NULL LIMIT 10000
);
```

## Data migrations

Data migrations transform existing data. Keep them separate from schema migrations:

```
WHY SEPARATE:
- Schema migrations can be reversed with DDL
- Data migrations may be irreversible (data loss)
- Data migrations can take hours on large tables
- Schema changes should be fast and atomic

PATTERN:
1. Schema migration: Add the new column/structure
2. Data migration: Transform the data
3. Schema migration: Add constraints, remove old structure
```

## Framework-specific guides

### Prisma

```prisma
// 1. Modify schema.prisma
// 2. Generate migration: npx prisma migrate dev --name description
// 3. Review the generated SQL before applying
// 4. For production: npx prisma migrate deploy
// 5. Rollback: npx prisma migrate resolve --rolled-back "migration_name"
```

### Knex

```js
exports.up = async function(knex) {
  await knex.schema.alterTable('users', (table) => {
    table.string('full_name');
  });
  // Data migration in same file is OK for small tables
  await knex('users').update({ full_name: knex.ref('name') });
};

exports.down = async function(knex) {
  await knex.schema.alterTable('users', (table) => {
    table.dropColumn('full_name');
  });
};
```

### Django

```python
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [('app', '0001_initial')]

    operations = [
        migrations.AddField('User', 'full_name', models.CharField(max_length=255, null=True)),
        # Separate data migration:
        migrations.RunPython(set_full_names, reverse_full_names),
    ]
```

## Rollback strategies

| Change type | Rollback method | Data loss risk |
|-------------|----------------|----------------|
| Add column | DROP COLUMN | None (column was new) |
| Add table | DROP TABLE | None (table was new) |
| Add index | DROP INDEX | None |
| Remove column | Re-add column (data lost) | **High** — need backup |
| Remove table | Recreate table (data lost) | **High** — need backup |
| Rename column | Rename back | Low (if both names exist) |
| Change type | Change back | **Medium** — may lose precision |
| Add constraint | DROP CONSTRAINT | Low |
| Remove constraint | Re-add constraint | Low |

**Rule:** If rollback causes data loss, keep a backup before the migration. For critical data, test the rollback on a staging copy of production first.

## Quality checklist

- [ ] Every migration has a reversible down/rollback path
- [ ] Schema changes and data migrations are in separate files
- [ ] Large table operations use concurrent/non-locking strategies
- [ ] Backfill operations are batched (not single UPDATE on millions of rows)
- [ ] Destructive changes use the expand-then-contract pattern
- [ ] Application code works with both old and new schema during transition
- [ ] Migrations tested up AND down on a staging copy of production data
- [ ] Destructive changes (DROP COLUMN, DROP TABLE) are documented in migration comments
- [ ] Foreign key constraints are added after data is consistent (not during backfill)
- [ ] Index creation uses CONCURRENTLY or equivalent for large tables

## Anti-patterns I avoid

- Mixing schema changes and data transforms in one migration — they have different rollback characteristics
- Running a single UPDATE on millions of rows — it locks the table for minutes
- Dropping a column in the same deploy that removes code references — if the deploy fails, the column is gone but the code still expects it
- Adding a NOT NULL column with a default on a large table — PostgreSQL rewrites every row; use ADD COLUMN nullable, SET DEFAULT, backfill, then SET NOT NULL
- Creating an index without CONCURRENTLY on a large table — it locks writes until the index builds
- Skipping the rollback test — if you can't roll it down, you can't deploy it safely
- Deploying schema + application changes simultaneously — deploy schema first, then application, then cleanup schema