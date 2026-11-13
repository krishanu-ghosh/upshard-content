---
title: "Indexes, Views, and Stored Procedures"
description: How a database index turns a table scan into a tree lookup, what views and stored procedures are for, and the DDL/DML/DQL/DCL split of SQL itself.
roadmap: system-design
node: indexes-views-and-stored-procedures
group: data-modeling-and-storage
video:
  provider: youtube
  id: "5t1fW3KG920"
  start: 0
  end: null
creator:
  name: developedbyed
  channel_url: https://www.youtube.com/@developedbyed
duration_seconds: 1170
level: intermediate
prerequisites:
  - acid-transactions-and-the-cap-theorem
takeaways:
  - A database index is a separate, sorted data structure — usually a B-tree — that lets the engine find matching rows without scanning the whole table, at the cost of extra work and storage on every write.
  - A view is a saved query that behaves like a table when you read from it, letting you hide complexity or restrict access without duplicating data.
  - A stored procedure is precompiled logic that lives in the database itself, reducing round trips and keeping business rules close to the data they protect.
  - SQL statements fall into four categories by what they do — DDL changes structure, DML changes data, DQL reads data, and DCL changes permissions — and knowing which is which clarifies what a statement is actually allowed to touch.
  - An index speeds up the exact lookups it was built for and nothing else; an index on the wrong column, or too many indexes on a write-heavy table, can make a database slower overall.
check_questions:
  - q: Why does adding an index speed up reads but slow down writes?
    a: An index is its own data structure that has to stay in sync with the table, so every INSERT, UPDATE, or DELETE must also update every index on that table, in exchange for SELECT queries being able to skip a full table scan.
  - q: What's the practical difference between a view and a regular table from the perspective of someone querying it?
    a: Almost none — a view is queried with the same SELECT syntax as a table — but a view has no storage of its own; it re-runs its underlying query (or reads a cached result) each time it's used.
  - q: Which SQL category does `GRANT SELECT ON orders TO analyst_role` belong to, and why?
    a: DCL (Data Control Language), because it changes who is permitted to do what, rather than changing the table's structure (DDL), its data (DML), or just reading data (DQL).
---

## Why a lookup can be fast or slow

Without help, finding a row by a non-key column means the database checks every row in the table — a full table scan. On a table with ten rows that's instant; on a table with ten million rows, it's the difference between a query that returns in milliseconds and one that takes seconds.

An **index** is a separate structure the database maintains alongside the table, built specifically to make one kind of lookup fast. Most relational databases implement a standard index as a **B-tree** — the same balanced, multi-way tree from the data structures lessons, where every leaf sits at the same depth and a lookup, insert, or range scan all cost `O(log n)`. Ordering `orders` by `customer_id` in a B-tree index means "find all orders for customer X" becomes a tree descent instead of a full scan.

```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
```

That index speeds up exactly one thing: lookups and range scans on `customer_id`. It does nothing for a query filtering on `order_date` unless a separate index covers that column too. And every index has a cost on the other side: each `INSERT`, `UPDATE`, or `DELETE` on `orders` now has to update the index as well as the table, so a table with ten indexes on it can be noticeably slower to write to than one with two. Indexing is a deliberate tradeoff, not a free speedup — index the columns your actual queries filter and sort by, not every column that looks useful.

## Views: a saved query that looks like a table

A **view** doesn't store data of its own — it's a named, saved `SELECT` statement that you can query as if it were a table:

```sql
CREATE VIEW high_value_orders AS
SELECT id, customer_id, total
FROM orders
WHERE total > 500;

SELECT * FROM high_value_orders WHERE customer_id = 'u_8f2a';
```

Views are useful for two different reasons: hiding complexity (a view can pre-join several tables so analysts query one simple name instead of repeating a five-table join) and restricting access (grant a role access to a view that only exposes certain columns, instead of the underlying table with everything in it).

## Stored procedures: logic that lives in the database

A **stored procedure** is a block of logic — conditionals, loops, multiple statements — saved and precompiled inside the database itself, callable by name:

```sql
CREATE PROCEDURE apply_discount(IN order_id INT, IN pct DECIMAL)
BEGIN
  UPDATE orders SET total = total * (1 - pct / 100) WHERE id = order_id;
END;

CALL apply_discount(42, 10);
```

Calling a stored procedure sends one round trip instead of however many statements the equivalent application code would send one at a time, and because the logic lives next to the data, the same validation applies no matter which application or script calls it. The tradeoff is that business logic written in a database-specific procedural dialect is harder to test, version, and move between database engines than the equivalent logic in application code — most teams reserve stored procedures for logic that genuinely benefits from living next to the data, not as a default place to put business rules.

## The four categories SQL statements fall into

Every SQL statement does one of four kinds of things, and the categories have names:

- **DDL (Data Definition Language)** — changes structure: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`.
- **DML (Data Manipulation Language)** — changes data: `INSERT`, `UPDATE`, `DELETE`.
- **DQL (Data Query Language)** — reads data: `SELECT`. (Some references fold this into DML; treating it separately is useful because read permissions and write permissions are usually granted independently.)
- **DCL (Data Control Language)** — changes permissions: `GRANT`, `REVOKE`.

Knowing which category a statement falls into is more than trivia — it's exactly the boundary most database permission systems are built around. A role granted DML access can insert and update rows but can't restructure the table; a role granted only DQL can read but never write at all.
