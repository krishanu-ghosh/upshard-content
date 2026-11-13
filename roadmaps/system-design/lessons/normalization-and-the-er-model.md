---
title: "Modeling Data: the Entity-Relationship Model and Normalization"
description: How to draw entities, attributes, and relationships, then normalize a schema through the first three normal forms — and when to break that rule.
roadmap: system-design
node: normalization-and-the-er-model
group: data-modeling-and-storage
video:
  provider: youtube
  id: "rBPQ5fg_kiY"
  start: 0
  end: null
creator:
  name: techTFQ
  channel_url: https://www.youtube.com/@techTFQ
duration_seconds: 2450
level: beginner
prerequisites:
  - sql-vs-nosql-and-data-modeling
takeaways:
  - "An entity-relationship diagram models the real-world things a database tracks (entities), the facts about them (attributes), and how they connect to each other (relationships), before a single table is created."
  - "Cardinality — one-to-one, one-to-many, many-to-many — determines whether a relationship is a foreign key on one side or needs its own junction table."
  - "Normalization is a sequence of rules, each one removing a specific kind of redundancy or anomaly; First Normal Form removes repeating groups, Second removes partial dependencies, Third removes transitive dependencies."
  - "An update anomaly — where the same fact is duplicated across rows and can drift out of sync — is the concrete problem normalization exists to prevent."
  - "Denormalization is normalization's intentional opposite: duplicating data back in for read speed, chosen deliberately once the cost of extra joins is measured, not as a shortcut for skipping the modeling step."
check_questions:
  - q: Why does a many-to-many relationship need its own table, unlike a one-to-many relationship?
    a: A foreign key column can only point at one row, so it can represent "many orders belong to one customer," but a relationship where many rows on both sides connect to many rows on the other needs a junction table holding one row per pairing.
  - q: What specific problem does moving a table from First to Second Normal Form fix?
    a: It removes partial dependencies — columns that depend on only part of a composite primary key — which otherwise force the same fact to be repeated on every row that shares that partial key, risking it going out of sync.
  - q: When is deliberately denormalizing a schema the right call?
    a: A reporting dashboard that reads the same aggregated totals thousands of times a second can justify storing a precomputed total alongside the raw rows, trading a small risk of staleness for avoiding a heavy join or aggregation on every read.
---


## Drawing the data before building it

An entity-relationship (ER) model is how you plan a database's shape before writing a single `CREATE TABLE`. It has three parts:

- **Entities** — the things being tracked: a `Customer`, an `Order`, a `Product`.
- **Attributes** — facts about an entity: a `Customer` has a `name` and an `email`.
- **Relationships** — how entities connect: a `Customer` *places* an `Order`; an `Order` *contains* `Product`s.

The relationship's **cardinality** decides how it gets built. A customer can place many orders, but each order belongs to exactly one customer — a one-to-many relationship, implemented as an `order.customer_id` foreign key pointing back at `customers`. An order can contain many products, and a product can appear on many orders — a many-to-many relationship, which needs its own junction table, `order_items`, with one row per (order, product) pairing and its own attributes like `quantity`.

## Normal forms: removing redundancy on purpose

Normalization is a sequence of rules for restructuring tables so the same fact is stored in exactly one place. Each "normal form" fixes a specific kind of problem left by the one before it.

Imagine a single wide table that tries to hold everything:

| order_id | customer_name | customer_email | product_name | product_price | quantity |
|---|---|---|---|---|---|
| 1 | Priya Shah | priya@example.com | Keyboard | 45.00 | 2 |
| 1 | Priya Shah | priya@example.com | Mouse | 20.00 | 1 |
| 2 | Priya Shah | priya@example.com | Monitor | 180.00 | 1 |

**First Normal Form (1NF)** requires atomic values and no repeating groups — this table already satisfies that, since each cell holds one value. But look at the redundancy: `customer_name` and `customer_email` repeat on every row for the same customer, and if Priya updates her email, every row that mentions her has to be updated too, or the data silently disagrees with itself. That's an **update anomaly**.

**Second Normal Form (2NF)** removes *partial dependencies* — a problem that shows up when a table has a composite key (here, effectively `order_id` + `product_name`) and some column depends on only part of it. `customer_name` depends only on `order_id` (via the customer who placed it), not on the product at all, so it's pulled into its own `orders` table.

**Third Normal Form (3NF)** removes *transitive dependencies* — a column that depends on another non-key column rather than on the key directly. `product_price` depends on `product_name`, not on the order, so it belongs in its own `products` table, referenced by id.

After normalizing, the single wide table becomes four: `customers`, `orders`, `products`, and `order_items` — each fact (a customer's email, a product's price) stored exactly once, referenced everywhere it's needed.

## Why this matters beyond tidiness

The payoff isn't aesthetic. An un-normalized schema is a schema where facts can quietly contradict themselves: update a product's price in one row and miss another, and now two orders disagree about what the product cost. Normalization makes that class of bug structurally impossible — there's only one row to update.

## When to break the rule on purpose

Normalization optimizes for write correctness, not read speed — every normalized query that needs the full picture has to join several tables back together, and joins cost time. **Denormalization** is the deliberate, measured reversal: storing a customer's name directly on an `orders` row anyway, for example, so a reporting query that scans a million orders doesn't have to join against `customers` for every single one. The difference between denormalizing and just skipping the modeling step is that denormalization happens *after* you understand the normalized shape and the specific read pattern that justifies duplicating data back in — not instead of doing that work at all.
