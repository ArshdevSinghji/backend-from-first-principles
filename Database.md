# Mastering Databases with Postgres

## 1. Fundamental Concepts

- Persistence is the main purpose of a database: data must survive after the process stops.
- RAM (primary memory) is fast but volatile and expensive.
- Disk (secondary memory) is slower but persistent and cost-effective for large volumes.
- Databases are preferred over text files because text files are harder to parse at scale, lack schema guarantees, and create concurrency issues under multi-user writes.

## 2. Choosing a Database

- Relational databases (SQL) use tables with strict schemas and are strong for data integrity.
- Non-relational databases (NoSQL) use flexible schemas and are useful for dynamic or unstructured data.
- Postgres is a strong default choice because:
  - It is open source and SQL-standard compliant.
  - It supports JSON and JSONB, enabling document-style data inside a relational system.

## 3. Postgres Data Types and Best Practices

- Integer family:
  - Serial and BigSerial: auto-incrementing identifiers.
  - SmallInt, Integer, BigInt: choose by expected numeric range.
- Decimal vs float:
  - Decimal/Numeric: exact precision, recommended for money.
  - Float/Real: approximate values, better for scientific calculations.
- String types:
  - Char(n): fixed length and padded, usually avoided.
  - Varchar(n): variable length with a limit.
  - Text: variable length without a practical limit; usually preferred in Postgres.
- Other important types:
  - UUID: safer IDs for distributed systems.
  - JSONB: preferred over JSON for indexing and query performance.
  - Enum: constrained value sets for stronger integrity.

## 4. Database Migrations

- Migrations are version control for schema changes.
- Up migration applies a change (for example, create table).
- Down migration reverts a change (for example, drop table).
- They keep developer and deployment environments consistent over time.

## 5. Data Modeling Relationships

- Naming conventions:
  - Use plural table names such as users and projects.
  - Use snake_case for column names such as full_name.
- One-to-one:
  - Example: users and user_profiles.
  - Profile table references user ID, often as both primary key and foreign key.
- One-to-many:
  - Example: one project has many tasks.
  - tasks table stores project_id.
- Many-to-many:
  - Example: users and projects.
  - Use a linking table such as project_members.
  - Composite key commonly combines user_id and project_id.

## 6. Constraints and Integrity

- Primary key implies uniqueness and non-null.
- Foreign key guarantees referenced records exist.
- Check constraints enforce custom validation rules at database level.
- Referential actions on delete:
  - Restrict blocks delete if dependent records exist.
  - Cascade removes dependent records automatically.

## 7. Performance and Security

- Parameterized queries:
  - Never build SQL with string concatenation.
  - Use placeholders to prevent SQL injection.
- Indexes:
  - Speed up reads on frequently filtered or joined columns.
  - Common targets: WHERE, JOIN, and ORDER BY columns.
  - Tradeoff: faster reads, slightly slower writes due to index maintenance.
- Triggers:
  - Automate database-side behavior.
  - Common example: auto-update updated_at on row modification.

## 8. API Query Design with SQL

- Pagination:
  - Use LIMIT and OFFSET for list endpoints.
- Filtering:
  - Use ILIKE for case-insensitive text matching.
- Joins:
  - Use LEFT JOIN when main records should be returned even without related rows.
