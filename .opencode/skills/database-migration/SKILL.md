---
name: database-migration
description: Safe schema changes and data migrations with rollback strategies and zero-downtime deployment
---

## What I do

I handle database schema and data migrations safely:

- Design reversible schema changes
- Write data migration scripts for transforming existing data
- Ensure zero-downtime deployment compatibility
- Plan rollback strategies for every change

## When to use me

Use this skill when:
- Adding, removing, or altering tables and columns
- Changing data types or constraints
- Migrating data between schemas
- Renaming tables or columns without data loss
- Adding indexes to large tables without locking
- Fixing schema drift between environments

## How I work

1. **Analyze the current schema** — Read migration files, model definitions, and schema docs. Understand the current state.
2. **Plan the migration** — Determine whether the change requires one step or multiple. Prefer expand-then-contract for production safety.
3. **Write the migration** — Use the project's migration tool (Alembic, Django, Prisma, Knex, Flyway, etc.). Follow existing naming conventions.
4. **Include a rollback** — Every migration must have a down/revert path. Test it.
5. **Handle data migrations separately** — Don't mix schema changes and data transforms in one migration. Create separate migrations for data transforms.
6. **Consider production impact** — For large tables: add columns as nullable first, backfill in batches, then add not-null constraints. Create indexes concurrently where supported.
7. **Verify** — Run migrations up and down. Check that the application still works with both old and new schemas during transition.

## Multi-step migration pattern

For risky changes (renaming columns, changing types, removing columns):

1. **Expand**: Add new column/table alongside old one. Dual-write to both.
2. **Migrate**: Backfill existing data from old to new.
3. **Contract**: Remove old column/table.

This allows zero-downtime deployment because the application works at every step.

## Guidelines

- Always provide a rollback path
- Never mix schema changes and data transforms in one migration
- Use transactions where supported; for large data migrations, use batched transactions
- Test migrations on a copy of production data when possible
- Document destructive changes (DROP, DELETE) clearly in the migration file