---
title: SQL for Data Analysis
description: "Writing the SQL an analyst actually uses day to day: GROUP BY aggregations, joins across tables, and window functions for running totals and ranks."
roadmap: ai-data-scientist
node: sql-for-data-analysis
group: data-science-coding
video:
  provider: youtube
  id: "QKIGsShyEsQ"
  start: 0
  end: null
creator:
  name: Luke Barousse
  channel_url: https://www.youtube.com/@LukeBarousse
duration_seconds: 23176
level: beginner
prerequisites: [python-for-data-science]
takeaways:
  - "GROUP BY collapses many rows into one per group, and every column you SELECT alongside it must be either in the GROUP BY list or wrapped in an aggregate like SUM() or COUNT()."
  - "An INNER JOIN drops rows with no match on the other side; a LEFT JOIN keeps every row from the left table and fills unmatched columns with NULL — picking the wrong one silently loses or duplicates data."
  - "Window functions like `RANK() OVER (PARTITION BY ...)` compute a value per row while still seeing every row in its group, which GROUP BY alone cannot do."
  - A query that runs without error can still be wrong; checking row counts before and after a join is the single most useful sanity check in analytical SQL.
check_questions:
  - q: "Why does SELECT customer_id, SUM(amount) FROM orders GROUP BY customer_id work, but adding order_date to the SELECT list without changing GROUP BY fails?"
    a: "GROUP BY collapses many rows into one per customer_id, so any extra column has to either be aggregated (like SUM(amount)) or added to the GROUP BY list itself, because the database wouldn't know which of the many order_date values to show otherwise."
  - q: "You LEFT JOIN orders to customers and row counts go up compared to just querying orders alone. What likely happened?"
    a: "The join key on the customers side probably isn't unique, so each order is matching more than one customer row, which duplicates rows; the fix is checking for and deduplicating the join key or adding a tighter join condition."
  - q: "What can a window function do that a plain GROUP BY cannot?"
    a: "A window function can return a value computed over a group of rows, such as a rank or running total, while keeping every individual row in the output, whereas GROUP BY collapses those rows down to one per group and loses the detail."
deeper:
  - label: SQL Window Functions Basics (Visually Explained)
    url: "https://www.youtube.com/watch?v=o666k19mZwE"
related_formulas: []
---

## GROUP BY: collapsing rows into answers

Most analytical questions take the shape "for each X, what's the total/average/count of Y?" That's exactly what `GROUP BY` does. `SELECT city, COUNT(*) FROM customers GROUP BY city` collapses every row sharing the same city into a single output row and counts how many went into each bucket. The rule that trips up beginners is this: once you group, every column in your `SELECT` list has to be either the grouping column itself or wrapped in an aggregate function (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`). The database physically can't show you `order_date` next to a `SUM(amount)` grouped by `customer_id`, because a single customer has many order dates — it has no single value to put there unless you tell it how to collapse those too, for example `MAX(order_date)` for "their most recent order."

`HAVING` is the filter that runs after grouping, as opposed to `WHERE`, which filters before. `SELECT city, COUNT(*) FROM customers GROUP BY city HAVING COUNT(*) > 100` keeps only cities with more than 100 customers — you couldn't write that condition in a `WHERE` clause because `COUNT(*)` doesn't exist until the grouping has happened. A common mistake is trying to filter on an aggregate with `WHERE`, which fails or gets rejected depending on the engine; if you're filtering on something computed by `SUM`, `COUNT`, or similar, it belongs in `HAVING`.

## Joins: combining tables without losing or duplicating rows

Real analysis questions usually span more than one table — orders live in one table, customers in another, products in a third. A join combines them on a shared key: `SELECT o.id, c.name FROM orders o JOIN customers c ON o.customer_id = c.id`. The join type controls what happens to rows that don't find a match. An `INNER JOIN` (the default for plain `JOIN`) keeps only rows where both sides match, silently dropping orders whose customer record was deleted or never existed. A `LEFT JOIN` keeps every row from the left table regardless of a match, filling in `NULL` for the right table's columns when nothing matches — use this when you want every order even if customer data is missing.

The failure mode to watch for is accidental row duplication. If the join key isn't unique on the right-hand table — say, customers has two rows for the same `customer_id` because of a data entry bug — every matching order row gets duplicated once per matching customer row. Your order totals will look inflated and nothing will have thrown an error. The habit that catches this: run `SELECT COUNT(*) FROM orders` before the join and compare it to the row count after the join. If they don't match and you expected an inner join to only filter (not multiply), go check whether your join key is actually unique on both sides, typically with `SELECT customer_id, COUNT(*) FROM customers GROUP BY customer_id HAVING COUNT(*) > 1`.

## Window functions: per-row answers that still see the group

`GROUP BY` forces a trade-off: you get the aggregate, but you lose the individual rows. Window functions give you both. `SELECT customer_id, amount, RANK() OVER (PARTITION BY customer_id ORDER BY amount DESC) AS rank FROM orders` returns every order row, untouched, with an added column ranking that order against only the other orders from the same customer. `PARTITION BY` is doing the same conceptual job as `GROUP BY` — splitting rows into buckets — but instead of collapsing each bucket to one row, the window function computes something while keeping the whole bucket visible.

This unlocks a family of queries that are awkward or impossible with `GROUP BY` alone. A running total: `SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date)` gives each row the cumulative sum of that customer's orders up to and including that date. "Top 3 per group" is a two-step pattern: compute `ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC)` in a subquery or common table expression, then filter `WHERE row_num <= 3` in the outer query — you can't filter directly on a window function in the same `SELECT` it's defined in, which is a frequent source of "why won't this `WHERE` clause work" confusion. Month-over-month comparison uses `LAG(amount) OVER (ORDER BY month)` to pull the previous row's value into the current row, so you can subtract them directly.

## Reading and sanity-checking a query before trusting it

SQL's biggest trap is that a syntactically valid query returning a plausible-looking table doesn't mean the query is correct. Before trusting a result, check three things: does the row count match what you'd expect given the join type you used; do a couple of groups add up by hand against a filtered, ungrouped version of the same data; and are there `NULL` values you didn't account for, since `NULL` doesn't equal anything in SQL (including another `NULL`), which silently excludes rows from equality filters and joins that look like they should match. Reading a query in execution order rather than written order also helps when debugging: conceptually, `FROM`/`JOIN` happens first, then `WHERE`, then `GROUP BY`, then `HAVING`, then `SELECT`, then `ORDER BY` — knowing that order explains why you can't reference a `SELECT` alias in a `WHERE` clause but can in an `ORDER BY` clause.
