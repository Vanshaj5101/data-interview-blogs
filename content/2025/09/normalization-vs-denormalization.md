---
title: "Normalization vs Denormalization"
slug: "normalization-vs-denormalization"
date: 2025-09-13
tags: ["database", "design", "sql"]
summary: "Interview-oriented explanation of Normalization vs Denormalization."
draft: false
---

```markdown
# Normalization vs Denormalization

## TL;DR

- **Normalization** minimizes redundancy by organizing data across multiple related tables.
- **Denormalization** combines related data to reduce join complexity and improve read performance.
- Use **normalization** in write-heavy OLTP systems to ensure data consistency.
- Use **denormalization** in read-heavy OLAP systems for faster analytics querying.

## What It Is & Why It Matters

In relational database design, how data is structured directly impacts system performance, data integrity, and maintainability. **Normalization** and **denormalization** are two opposing strategies:

- **Normalization** reduces data redundancy and ensures consistency by splitting data into well-structured related tables, following defined normal forms.
- **Denormalization** improves query performance by combining information into fewer tables, accepting some redundancy for speed and simplicity.

Choosing between these strategies—or applying them selectively—is critical to building scalable and maintainable systems, especially in environments where read/write patterns vary significantly.

## Step-by-Step Explanation

### Normalization

Normalization organizes data to minimize repetition and preserve data correctness. This is achieved through "normal forms." The first three are the most commonly applied:

1. **First Normal Form (1NF)**:
   - All columns must contain atomic (indivisible) values.
   - Each row must be uniquely identifiable via a primary key.

2. **Second Normal Form (2NF)**:
   - Meets 1NF.
   - Removes partial functional dependencies; every non-key column must depend on the whole primary key (not just part of a composite primary key).

3. **Third Normal Form (3NF)**:
   - Meets 2NF.
   - Removes transitive dependencies; non-key attributes must not depend on other non-key attributes.

Normalization reinforces integrity with foreign keys and constraints but often increases join requirements in queries.

### Denormalization

Denormalization starts with a normalized schema and reintroduces redundancy when query performance is a priority. Common techniques include:

- Merging tables to reduce join costs.
- Storing aggregated or derived values.
- Caching frequently queried results.

Denormalization is a deliberate optimization strategy, not a disregard for data quality. It requires careful update management and typically complements advanced querying needs.

## Schema Examples (Valid for PostgreSQL)

### Normalized Schema

Consider a database tracking customer orders:

```sql
-- customers table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

-- orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date DATE NOT NULL
);

-- order_items table
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(order_id),
    product_name TEXT NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0)
);
```

This setup follows 3NF: distinct entities (customers, orders, items) stored in separate tables, with relationships expressed via foreign keys.

### Denormalized Schema

To speed up reporting or simplify analytics, denormalized structures can summarize related data:

```sql
-- summary table with embedded customer and order item details
CREATE TABLE order_summary (
    order_id INTEGER PRIMARY KEY,
    customer_name TEXT NOT NULL,
    email TEXT,
    order_date DATE NOT NULL,
    product_list TEXT,
    total_quantity INTEGER
);
```

Here, `product_list` might be a comma-separated string like `Widget A (2), Widget B (4)`, and `total_quantity` sums item quantities per order. These fields are derived and stored in advance to avoid joins during common queries.

### Populating the Denormalized Table (Python + SQL)

You might use periodic ETL jobs (e.g., written in Python) to populate this table:

```python
import psycopg2

conn = psycopg2.connect(dbname="mydb", user="myuser", password="mypassword")
cur = conn.cursor()

cur.execute("""
    INSERT INTO order_summary (order_id, customer_name, email, order_date, product_list, total_quantity)
    SELECT 
        o.order_id,
        c.name,
        c.email,
        o.order_date,
        string_agg(oi.product_name || ' (' || oi.quantity || ')', ', ') AS product_list,
        SUM(oi.quantity) AS total_quantity
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY o.order_id, c.name, c.email, o.order_date
    ON CONFLICT (order_id) DO UPDATE
    SET 
        product_list = EXCLUDED.product_list,
        total_quantity = EXCLUDED.total_quantity;
""")

conn.commit()
cur.close()
conn.close()
```

Note: `ON CONFLICT` handles upserts if order IDs already exist, ensuring the summary remains current.

## Performance Considerations

| Factor                 | Normalized Design                     | Denormalized Design                          |
|------------------------|----------------------------------------|----------------------------------------------|
| **Disk I/O**           | Higher, due to frequent joins          | Lower, as data is pre-joined                 |
| **Insert/Update Speed**| Slower (multiple tables)              | Faster reads, but updates need careful logic |
| **Data Integrity**     | Strong—relies on constraints           | Weaker—requires managed redundancy           |
| **Storage Usage**      | Efficient                              | Larger, due to duplication                   |
| **Indexing**           | Relational join-centric indexes        | Often uses multicolumn or partial indexes    |
| **Query Simplicity**   | More joins, more complex SQL           | Simpler and faster queries                   |
| **Scalability**        | Well-suited to OLTP workloads          | Common in OLAP/reporting databases           |

Indexes in normalized schemas typically target foreign keys for join performance. In denormalized schemas, indexes should support aggregate metrics and filter conditions.

## Common Pitfalls

- ❌ Over-normalizing without analyzing access patterns—leading to excessive joins.
- ❌ Denormalizing without rigor—causing update anomalies and inconsistencies.
- ❌ Ignoring query workload when designing schemas.
- ❌ Neglecting to add or adjust indexes after schema transitions.
- ❌ Using denormalization as a shortcut in place of a proper ETL or caching solution.

## Quick Quiz

1. **What is the primary trade-off between normalization and denormalization?**
   <details>
   <summary>Answer</summary>
   Normalization ensures data consistency but may slow reads due to joins. Denormalization improves read performance by storing combined data, but at the risk of redundancy and write anomalies.
   </details>

2. **Which normal form removes transitive dependencies?**
   <details>
   <summary>Answer</summary>
   Third Normal Form (3NF).
   </details>

3. **In PostgreSQL, what construct can store precomputed, denormalized summaries efficiently?**
   <details>
   <summary>Answer</summary>
   Materialized views.
   </details>

## References

- [PostgreSQL Documentation: Data Definition](https://www.postgresql.org/docs/current/sql-createtable.html)
- [Database Normalization – Microsoft Docs](https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization)
- [PostgreSQL Materialized Views](https://www.postgresql.org/docs/current/sql-creatematerializedview.html)
```