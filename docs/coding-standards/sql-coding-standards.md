# SQL Coding Standards

These standards apply to all SQL written across EWB projects, including raw queries, stored procedures, migrations, and database schema definitions.

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Table | snake_case, plural | `orders`, `order_items` |
| Column | snake_case | `created_at`, `customer_id` |
| Primary key | `id` | `id` |
| Foreign key | `{referenced_table_singular}_id` | `customer_id`, `order_id` |
| Index | `idx_{table}_{column(s)}` | `idx_orders_customer_id` |
| Unique constraint | `uq_{table}_{column(s)}` | `uq_users_email` |
| Foreign key constraint | `fk_{table}_{referenced_table}` | `fk_orders_customers` |
| Stored procedure | snake_case, descriptive verb | `get_orders_by_customer` |
| View | snake_case, prefixed with `vw_` | `vw_active_orders` |

---

## Formatting

- SQL keywords in **UPPERCASE**: `SELECT`, `FROM`, `WHERE`, `JOIN`, `INSERT`, `UPDATE`, `DELETE`
- Table and column names in **lowercase**
- Indent joins and conditions to align with the main clause
- Place each selected column on its own line for queries with more than two columns

```sql
SELECT
    o.id,
    o.status,
    o.created_at,
    c.email AS customer_email
FROM
    orders o
    INNER JOIN customers c ON c.id = o.customer_id
WHERE
    o.status = 'pending'
    AND o.created_at >= '2025-01-01'
ORDER BY
    o.created_at DESC;
```

---

## Schema Design

- Every table must have a primary key
- Use `UUID` or `BIGINT GENERATED ALWAYS AS IDENTITY` for primary keys — avoid composite primary keys as identifiers
- Include `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` on all tables
- Include `updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` on tables that are mutated after creation
- Apply `NOT NULL` constraints by default — use `NULL` explicitly and intentionally
- Use appropriate column types: `BOOLEAN` not `TINYINT`, `TEXT` not `VARCHAR(MAX)`, `TIMESTAMPTZ` not `TIMESTAMP`

---

## Indexes

- Add indexes on all foreign key columns
- Add indexes on columns frequently used in `WHERE` clauses or `ORDER BY`
- Avoid over-indexing — every index adds overhead to writes
- Use composite indexes strategically; column order matters (most selective first)
- Name all indexes explicitly — do not rely on auto-generated names

---

## Queries

- Always specify column names in `SELECT` — never use `SELECT *` in application code
- Always use parameterized queries — never concatenate user input into SQL strings (see [secure-coding-guidelines.md](../security/secure-coding-guidelines.md))
- Avoid `SELECT DISTINCT` as a shortcut for fixing duplicate data problems — fix the root cause
- Use `EXISTS` instead of `COUNT(*)` when checking for the presence of rows
- Prefer explicit `INNER JOIN` over implicit comma joins

---

## Migrations

- All schema changes must be applied via migration files — never modify the database schema manually in any environment
- Migration file naming: `V{version}__{description}.sql` (Flyway) or numeric prefix (Entity Framework, Liquibase)
- Migrations must be **idempotent** where possible
- Migrations must be **reversible** — include a rollback migration for every forward migration
- Never drop a column or table in the same migration that removes it from application code — allow a grace period
- Test migrations against a copy of production data before applying to production

---

## Stored Procedures and Functions

- Prefer application-layer logic over stored procedures for business rules — code in stored procedures is harder to test and version
- Use stored procedures only for bulk operations, reporting queries, or operations requiring database-level transactions
- Document stored procedures with a comment block describing inputs, outputs, and side effects

---

## Performance Considerations

- Use `EXPLAIN ANALYZE` to understand query execution plans before merging changes to performance-sensitive queries
- Avoid queries inside loops in application code — use batch queries or `IN` clauses
- Use pagination (`LIMIT`/`OFFSET` or keyset pagination) for large result sets
- Be aware of N+1 query problems in ORM usage — use eager loading (`Include` in EF Core, `JOIN FETCH` in JPA) where appropriate
