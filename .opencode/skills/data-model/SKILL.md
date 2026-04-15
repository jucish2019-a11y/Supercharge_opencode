---
name: data-model
description: Design database schemas, normalization, indexing strategy, and data access patterns
---

## What I do

I design data models and database schemas:

- Normalize or denormalize based on access patterns
- Design tables, columns, constraints, and relationships
- Plan indexing strategy for query performance
- Define data access patterns and repository interfaces

## When to use me

Use this skill when:
- Designing a new database schema from requirements
- Adding tables or relationships to an existing schema
- Resolving normalization/denormalization tradeoffs
- Planning indexes for a slow query or new access pattern
- Designing a data model before writing migrations

## How I work

1. **Understand the domain** — Identify entities, attributes, and relationships from requirements. List the questions the data must answer.
2. **Define access patterns** — Before designing tables, list how data will be read and written. Optimize the schema for the most frequent and latency-sensitive queries.
3. **Design the logical model** — Create the entity-relationship model. Determine cardinality (1:1, 1:N, N:M). Choose between normalization (less duplication, more joins) and denormalization (faster reads, more write complexity) based on access patterns.
4. **Map to physical schema** — Convert the logical model to tables, columns, types, and constraints. Use appropriate column types. Add foreign keys for referential integrity.
5. **Design indexes** — Add indexes for:
   - Primary keys (usually automatic)
   - Foreign key columns (for JOINs)
   - Columns used in WHERE clauses
   - Columns used in ORDER BY (compound indexes)
   - Unique constraints for business rules
6. **Define data access** — Create repository/query interfaces. Define the CRUD operations and custom queries needed.
7. **Plan for growth** — Consider partitioning, archiving, soft deletes, and audit columns (created_at, updated_at, deleted_at).

## Common patterns

- **N:M relationship**: Junction table with composite primary key
- **Soft delete**: `deleted_at` column instead of actual deletion
- **Multi-tenant**: `tenant_id` column with row-level security or filtering
- **Temporal data**: `valid_from`/`valid_to` for slowly changing dimensions
- **Polymorphic**: Either a type column + optional columns, or separate tables + a base table
- **Tree/hierarchy**: `parent_id` self-reference, or materialized path, or closure table

## Guidelines

- Always have primary keys
- Use foreign keys for referential integrity unless the ORM handles it
- Add NOT NULL constraints by default — make nullable an explicit decision
- Use appropriate column types (don't store numbers as strings, don't store JSON when columns work)
- Index for your queries, not for every column — indexes have write overhead
- Always include created_at and updated_at timestamps
- Consider soft deletes over hard deletes for auditable data