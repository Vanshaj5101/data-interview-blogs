---
layout: post
title: "Normalization vs Denormalization"
slug: "normalization-vs-denormalization"
date: 2025-09-14
tags: ["database", "design", "sql"]
summary: "Interview-oriented explanation of Normalization vs Denormalization."
permalink: /normalization-vs-denormalization/
---

## TL;DR

- **Normalization** reduces data redundancy by organizing data into related tables.
- **Denormalization** improves read performance by combining related tables, often with some redundancy.
- Use **normalization** for OLTP (transactional systems) and **denormalization** for OLAP (read-heavy systems).
- Choosing the right level of normalization depends on **query patterns** and **scaling needs**.

## What It Is & Why It Matters

Normalization and denormalization are foundational concepts in relational database design. They define how data is structured, influencing performance, maintainability, and data integrity.

- **Normalization** organizes data into related tables to reduce redundancy and improve consistency.
- **Denormalization** merges related tables, introducing redundancy to optimize read performance—especially important in analytical workflows.

Understanding when and how to apply these techniques is essential for designing robust database schemas, particularly in systems geared toward transactions (OLTP) or analytics (OLAP).

## Step-by-Step Explanation

### Normalization

Normalization organizes data through successive "normal forms" (NFs), each imposes stricter rules to eliminate different types of redundancy:

1. **First Normal Form (1NF)**: Ensure each column contains only atomic (indivisible) values; no repeating groups.
2. **Second Normal Form (2NF)**: Eliminate partial dependencies; every non-key column must depend on the full primary key.
3. **Third Normal Form (3NF)**: Eliminate transitive dependencies; non-key attributes must depend only on the primary key.

**Goals**:
- Minimize redundant data
- Facilitate consistent updates
- Improve data integrity

Use in systems where data accuracy and frequent writes are critical, such as transactional applications.

### Denormalization

Denormalization merges related tables to reduce the number of joins in queries. This introduces redundancy, but can significantly improve performance for read-intensive applications.

**Goals**:
- Optimize query performance
- Reduce JOIN costs
- Simplify reporting and aggregation logic

Use in analytics and reporting environments where read efficiency outweighs the cost of redundant data.

## Examples: SQL in PostgreSQL

### Normalization Example

Start with a single unnormalized table:

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name TEXT,
    customer_email TEXT,
    product_name TEXT,
    product_price NUMERIC
);
```

This structure leads to data duplication—multiple customer or product entries across orders. Normalize it:

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC NOT NULL
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id)
);
```

Each entity has its own table, and foreign keys maintain relationships. This improves data consistency and avoids storing duplicate values.

### Denormalization Example

To speed up read operations, especially for reporting, consolidate data into a single table:

```sql
CREATE TABLE orders_denorm (
    order_id SERIAL PRIMARY KEY,
    customer_name TEXT NOT NULL,
    product_name TEXT NOT NULL,
    product_price NUMERIC NOT NULL
);
```

Now, common queries like aggregation run faster:

```sql
SELECT product_name, SUM(product_price) AS total_sales
FROM orders_denorm
GROUP BY product_name;
```

This avoids costly joins and is useful for dashboards or business intelligence tools.

## Performance Considerations: Indexes, I/O, Scaling, Trade-Offs

### Normalization

**Pros**:
- Minimal data duplication
- Clear referential integrity
- More efficient updates and deletes

**Cons**:
- Queries are more complex; require more joins
- JOINs can be performance bottlenecks at scale
- Harder to optimize read-heavy use cases

### Denormalization

**Pros**:
- Faster read performance for analytical queries
- Simpler query logic
- Better suited for OLAP and data warehouse models

**Cons**:
- Redundant data increases storage
- Risk of data inconsistency from missed updates
- More complex write logic

### Indexes and Query Plans

In PostgreSQL:

- Indexes are helpful regardless of schema design, but are especially important for foreign key lookups in normalized schemas.
- Use `EXPLAIN ANALYZE` to observe join costs and overall query execution plans for performance tuning.
- **Materialized views** can bridge the gap—normalize for consistency, but cache denormalized structures for faster reads.

Example:

```sql
CREATE MATERIALIZED VIEW product_sales AS
SELECT
    p.name AS product_name,
    SUM(p.price) AS total_sales
FROM
    orders o
JOIN
    products p ON o.product_id = p.product_id
GROUP BY
    p.name;
```

Materialized views improve performance for frequently run aggregate queries, while maintaining normalized relationships behind the scenes.

## Common Pitfalls

- ❌ Over-normalizing without regard for query patterns
- ❌ Premature denormalization introduces inconsistent data
- ❌ Missing indexes on foreign keys in normalized tables
- ❌ Using denormalization for systems with frequent updates
- ❌ Failing to update all copies of a field in denormalized tables

## Quick Quiz

### 1. What’s a primary benefit of normalization?

<details>
<summary>Answer</summary>
To eliminate data redundancy and improve data integrity.
</details>

### 2. When is denormalization typically preferred?

<details>
<summary>Answer</summary>
In read-heavy systems like data warehouses where query performance is more important than data duplication.
</details>

### 3. What PostgreSQL feature can help combine benefits of both approaches?

<details>
<summary>Answer</summary>
Materialized views — they allow precomputed queries over normalized data for better read performance.
</details>

## References

- [PostgreSQL: Constraints and Normal Forms](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [PostgreSQL Wiki: Database Normalization](https://wiki.postgresql.org/wiki/Database_Normalization)