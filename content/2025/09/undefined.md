---
title: "undefined"
slug: "undefined"
date: 2025-09-13
tags: [""]
summary: "Interview-oriented explanation of undefined."
draft: false
---

```markdown
# Normalization vs Denormalization

## TL;DR

- **Normalization** reduces redundancy and improves data integrity by organizing data into multiple related tables.
- **Denormalization** consolidates data to reduce joins and improve read performance at the cost of redundancy.
- Use **normalization** for systems with frequent writes (OLTP), and **denormalization** for systems optimized for reads (OLAP).
- Real-world systems often employ a hybrid approach based on workload requirements.

## What It Is & Why It Matters

Normalization and denormalization are fundamental database design strategies that directly affect performance, maintainability, and data correctness.

- **Normalization** structures data across multiple related tables and applies a set of formal rules (normal forms) to reduce duplication and enforce data integrity.
- **Denormalization** merges related tables or introduces redundancy to simplify queries and improve read performance, particularly in analytics-heavy systems.

Design decisions around these approaches vary depending on workload patterns—read-heavy vs. write-heavy, transactional vs. analytical—and affect query complexity, data consistency, and overall scalability.

## Normalization: A Step-by-Step Guide

The goal of normalization is to eliminate data redundancy and ensure consistent updates.

1. **First Normal Form (1NF)**  
   All columns must contain only atomic values—no arrays, sets, or nested records.

2. **Second Normal Form (2NF)**  
   Eliminate partial dependencies—non-key columns must depend on the entire primary key (important in composite keys).

3. **Third Normal Form (3NF)**  
   Eliminate transitive dependencies—non-key columns must depend only on the primary key, not on other non-key columns.

4. **Boyce-Codd Normal Form (BCNF)**  
   Strengthens 3NF by ensuring every determinant is a candidate key.

5. **Higher Normal Forms (4NF and 5NF)**  
   These handle multi-valued and join dependencies. They're less common in transactional design but relevant in complex data domains.

Normalization improves long-term maintainability and prevents anomalies related to insertion, update, or deletion.

## Denormalization: When and How

Denormalization enhances performance by reducing the need to join multiple tables during queries. It involves:

1. Combining frequently joined tables into a single table.
2. Repeating data (e.g., customer name in multiple rows).
3. Precomputing and storing derived values such as totals or counts.
4. Using PostgreSQL features like materialized views or external caching for controlled redundancy.

This approach improves read performance, reduces query complexity, and simplifies reporting. However, it requires careful maintenance to avoid inconsistencies during updates.

## PostgreSQL Examples

### Normalized Schema

```sql
-- Customers table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

-- Orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date DATE NOT NULL
);

-- Order items table
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(order_id),
    product_name TEXT NOT NULL,
    quantity INTEGER NOT NULL
);
```

### Denormalized Table

```sql
-- Denormalized orders table
CREATE TABLE orders_denormalized (
    order_id SERIAL PRIMARY KEY,
    customer_name TEXT NOT NULL,
    customer_email TEXT NOT NULL,
    order_date DATE NOT NULL,
    product_name TEXT NOT NULL,
    quantity INTEGER NOT NULL
);
```

In the denormalized version, customer information and product data are duplicated for each row, reducing the need for joins.

### Query Comparison

With normalization:

```sql
SELECT c.name, o.order_date, oi.product_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id;
```

With denormalization:

```sql
SELECT customer_name, order_date, product_name
FROM orders_denormalized;
```

The denormalized query is simpler and faster for static datasets but sacrifices storage efficiency and consistency guarantees.

## Performance Considerations

| Factor                  | Normalization                            | Denormalization                         |
|------------------------|-------------------------------------------|------------------------------------------|
| Storage Efficiency     | High (minimal duplication)                | Lower (duplicate data increases size)    |
| Data Integrity         | High (via constraints and references)     | Lower (risk of inconsistencies)          |
| Read Performance       | May be slower (requires joins)            | Faster (fewer joins, flatter structure)  |
| Write Performance      | Slower (multiple inserts/updates)         | Faster in some cases, but complex logic needed to maintain consistency |
| Indexing               | Indexes on foreign keys and join columns  | May use composite or partial indexes to optimize large tables |
| Use Case Suitability   | OLTP (frequent writes)                    | OLAP (frequent reads)                    |

Indexing is crucial in either strategy. PostgreSQL supports B-tree indexes for equality and range queries, and GIN indexes for full-text search or array containment.

PostgreSQL’s materialized views offer a way to maintain denormalized snapshots that can be refreshed periodically:

```sql
CREATE MATERIALIZED VIEW customer_order_summary AS
SELECT c.name, COUNT(o.order_id) AS total_orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;

-- Refresh to update data
REFRESH MATERIALIZED VIEW customer_order_summary;
```

This lets you precompute and cache query results while keeping data normalized at the base level.

## Common Pitfalls

Avoid these common design issues:

- ❌ Over-normalizing for reporting or analytical systems, causing poor query performance.
- ❌ Denormalizing prematurely, leading to difficult update logic and redundant storage.
- ❌ Failing to implement triggers or synchronization logic to keep denormalized values in sync.
- ❌ Missing indexes, especially on keys used in joins or filters.
- ❌ Assuming one model fits all—design must match system workload.

## Quick Quiz

1. **Why might denormalization improve query performance?**  
   <details><summary>Answer</summary>
   It reduces the need for joins, enabling faster reads from a single flat table.
   </details>

2. **Which normal form eliminates transitive dependencies?**  
   <details><summary>Answer</summary>
   Third Normal Form (3NF).
   </details>

3. **What PostgreSQL feature helps implement query-based denormalization?**  
   <details><summary>Answer</summary>
   Materialized views.
   </details>

## References

- [PostgreSQL Documentation: Table Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [PostgreSQL Documentation: Materialized Views](https://www.postgresql.org/docs/current/rules-materializedviews.html)
```
```