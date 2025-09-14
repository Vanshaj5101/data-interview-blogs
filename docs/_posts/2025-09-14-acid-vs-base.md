---
layout: post
title: "ACID vs BASE"
slug: "acid-vs-base"
date: 2025-09-14
tags: ["database", "transactions", "theory"]
summary: "Interview-oriented explanation of ACID vs BASE."
permalink: /acid-vs-base/
---

## TL;DR

- **ACID** stands for Atomicity, Consistency, Isolation, Durability — key properties ensuring data integrity in traditional relational databases.
- **BASE** stands for Basically Available, Soft state, Eventually consistent — principles supporting availability and scalability in distributed NoSQL systems.
- ACID offers strong consistency at the cost of scalability limitations, while BASE systems prioritize availability and partition tolerance, potentially sacrificing immediate consistency.
- Choosing ACID or BASE depends on your application's requirements for consistency, availability, and performance.

## What it is & why it matters

In data engineering, understanding **ACID** and **BASE** models is crucial for designing systems that balance data consistency, performance, and fault tolerance. These models influence how databases structure transactions, handle failure recovery, and perform under load or in distributed environments.

**ACID** transactions are the standard for relational databases such as PostgreSQL. They ensure that all operations within a transaction are executed safely, maintaining the integrity of the data even in the face of errors or crashes.

**BASE** principles are common in distributed NoSQL systems such as Cassandra or DynamoDB. These systems often relax consistency guarantees to improve availability and fault tolerance, accepting temporary inconsistencies that eventually resolve.

Understanding these models is important for making informed architectural decisions, especially when building data pipelines or distributed applications.

## Step-by-step explanation

### ACID Transactions

1. **Atomicity**: Ensures that a series of operations within a transaction are all completed successfully, or none are applied.
2. **Consistency**: Guarantees that transactions bring the database from one valid state to another, satisfying database constraints.
3. **Isolation**: Prevents concurrent transactions from interfering with each other, maintaining correct execution order.
4. **Durability**: Once a transaction is committed, its results are permanently recorded, even in case of system failure.

### BASE Principles

1. **Basically Available**: The system remains operational and responds to requests, even during partial outages.
2. **Soft State**: The system’s state can change over time, even without external writes, due to eventual updates propagating through the system.
3. **Eventually Consistent**: While data may be temporarily inconsistent, it will become consistent over time, assuming no new updates.

### CAP Theorem Context

The **CAP theorem** explains key trade-offs in distributed systems:

- **Consistency (C)**: All nodes return the same data at the same time.
- **Availability (A)**: Every request receives a response.
- **Partition Tolerance (P)**: The system remains operational despite network failures that prevent communication between nodes.

Distributed systems must navigate trade-offs — no system can guarantee all three simultaneously. 

- ACID systems typically favor **Consistency and Availability** in single-node or tightly coupled environments.
- BASE systems often prioritize **Availability and Partition Tolerance**, accepting eventual consistency.

## Examples (SQL for PostgreSQL; Python if relevant)

### Example 1: ACID-compliant transaction in PostgreSQL

```sql
BEGIN;

INSERT INTO accounts(user_id, balance) VALUES (101, 500);
UPDATE accounts SET balance = balance - 100 WHERE user_id = 101;
UPDATE accounts SET balance = balance + 100 WHERE user_id = 102;

COMMIT;
```

If an error occurs during any part of the transaction, it can be reversed:

```sql
ROLLBACK;
```

This enforces **atomicity**, ensuring either all operations succeed or none are applied.

### Example 2: Conceptual example of eventual consistency (Python simulation)

```python
import time
from threading import Thread

db_replica = {"key": "old_value"}

def write_to_primary():
    time.sleep(1)  # Simulate network latency or sync delay
    db_replica["key"] = "new_value"

Thread(target=write_to_primary).start()

# Immediate read (before replica updated)
print(db_replica["key"])  # Outputs: old_value
time.sleep(2)
print(db_replica["key"])  # Outputs: new_value
```

This simulates a write arriving at a replica after some delay. Intermediate reads may return outdated values, demonstrating **eventual consistency**.

## Performance considerations

### ACID (Relational systems like PostgreSQL)

- **Indexes** improve read performance but can slow down writes and increase overhead during transaction processing.
- **Isolation levels** (`READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`) affect concurrency and speed — higher isolation reduces the chances of anomalies but can limit throughput.
- **Durability** uses techniques like `fsync` to ensure data is flushed to disk, which can constrain write performance.
- **Scalability** is typically vertical (scale-up), although horizontal approaches like read replicas and partitioning are possible with additional complexity.

### BASE (Distributed NoSQL systems)

- **Eventual consistency** supports high availability, low latency, and **horizontal scaling** across commodity servers.
- **Stale reads** can occur because replicas may not be fully in sync immediately after a write.
- **Query capabilities** vary — some systems require external indexing mechanisms or don't support complex joins.
- Write-heavy workloads benefit from BASE design due to its ability to distribute operations across nodes.

## Common pitfalls

- ❌ Believing that all NoSQL systems implement BASE — many offer tunable consistency models.
- ❌ Assuming ACID always delivers better performance — stronger isolation levels can degrade throughput.
- ❌ Using BASE systems for workloads requiring strict guarantees (e.g., financial ledgers).
- ❌ Ignoring the implications of write amplification and disk I/O in ACID systems.
- ❌ Over-architecting for consistency when availability and partition tolerance are more critical.

## Quick quiz (3 questions; answers hidden in `<details>` blocks)

**1. What does the "Isolation" property in ACID aim to prevent?**

<details>
<summary>Answer</summary>
Prevent concurrent transactions from interfering with each other, ensuring the results are as if transactions were executed sequentially.
</details>

---

**2. What is one core difference between ACID and BASE models?**

<details>
<summary>Answer</summary>
ACID emphasizes strong consistency, whereas BASE allows for eventual consistency and prioritizes availability.
</details>

---

**3. In PostgreSQL, what statement is used to ensure a group of queries behaves atomically?**

<details>
<summary>Answer</summary>
Use `BEGIN;` to start a transaction, `COMMIT;` to apply changes, or `ROLLBACK;` to undo them.
</details>