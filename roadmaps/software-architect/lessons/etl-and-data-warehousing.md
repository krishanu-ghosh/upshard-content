---
title: "ETL, ELT & the Data Warehouse"
description: Why operational databases and analytical warehouses are different systems, and how ETL and ELT move data between them.
roadmap: software-architect
node: etl-and-data-warehousing
group: data-and-integration-architecture
video:
  provider: youtube
  id: "G91ysmOC6bg"
  start: 0
  end: null
creator:
  name: Intellipaat
  channel_url: https://www.youtube.com/@Intellipaat
duration_seconds: 945
level: intermediate
prerequisites: []
takeaways:
  - "An operational (OLTP) database is built for many small, fast reads and writes from an application — one order, one row at a time — while a data warehouse is built for OLAP: scanning and aggregating millions of rows for a report or a dashboard."
  - "Running analytical queries directly against the operational database risks locking rows or slowing the application that customers are actually using, which is the real reason a separate warehouse exists, not just raw scale."
  - "ETL (extract, transform, load) cleans and reshapes data before it ever reaches the warehouse; ELT (extract, load, transform) dumps raw data in first and transforms it inside the warehouse using the warehouse's own compute."
  - "ELT became the default once cloud warehouses like BigQuery and Snowflake made on-demand compute cheap — transforming in SQL inside the warehouse is now usually simpler to operate than a separate transformation cluster."
  - "A star schema — one fact table of measurable events surrounded by dimension tables describing who, what, and when — is the classic warehouse data model, optimized for the kind of group-by-and-aggregate queries a dashboard runs, not for the normalized, update-friendly shape of an operational schema."
check_questions:
  - q: "Why can't a reporting dashboard just query the production database directly?"
    a: "A dashboard query scans and aggregates a large slice of the table, which competes for the same rows and locks the application's fast, small, transactional queries need — at real scale this can slow down or block the production workload, so the two concerns are separated into an operational database and a warehouse."
  - q: "What actually changes between ETL and ELT?"
    a: "The order of the transform and load steps — ETL transforms data in a separate pipeline before loading it into the warehouse, while ELT loads the raw data into the warehouse first and runs the transformation afterward as SQL inside the warehouse's own compute."
  - q: "Why is a star schema shaped differently from a normalized operational schema?"
    a: "An operational schema is normalized to avoid update anomalies on single-row writes; a star schema denormalizes on purpose, pre-joining a central fact table to its dimension tables so a BI query can aggregate millions of rows with few joins instead of many."
deeper:
  - label: "What is Data Pipeline? | Why Is It So Popular? (ByteByteGo)"
    url: "https://www.youtube.com/watch?v=kGT4PcTEPP8"
related_formulas: []
---

## Two databases, two jobs

Every system of real size ends up with at least two kinds of database, even if nobody designed it that way on purpose. The first is **operational** — the one your application reads and writes while a user is doing something: placing an order, updating a profile, checking a balance. These are OLTP (online transaction processing) workloads: many small, fast, highly concurrent queries, each touching a handful of rows, each needing to finish in milliseconds.

The second is **analytical** — the one someone queries to answer "how many orders did we ship last quarter, broken down by region and product line?" These are OLAP (online analytical processing) workloads: few queries, but each one scans and aggregates a huge slice of the data. Run that kind of query directly against the operational database and you're competing for the same rows, the same locks, and the same I/O that the paying customer's checkout request needs right now. At meaningful scale, that's not a performance nuisance — it's an outage waiting to happen. The architectural answer is to keep them as two separate systems: the operational database for transactions, and a **data warehouse** purpose-built for analytical queries, fed by a pipeline that copies and reshapes data from the first into the second.

## ETL vs. ELT — same three verbs, different order

The pipeline that moves data from operational sources into the warehouse goes by two names depending on when the transformation happens.

**ETL (extract, transform, load)** is the classic shape: pull data out of source systems, run it through a separate transformation stage — cleaning, filtering, joining, reshaping — and only then load the finished result into the warehouse. This made sense when warehouse compute was expensive and scarce; you did the heavy lifting somewhere cheaper first and only loaded the warehouse with data that was already in its final shape.

**ELT (extract, load, transform)** flips the order: raw data is loaded into the warehouse as-is, and the transformation happens afterward, as SQL (or a tool like dbt generating SQL) running inside the warehouse itself. This only became practical once cloud warehouses — BigQuery, Snowflake, Redshift — made on-demand, scale-to-zero compute cheap and easy to provision. Instead of operating a separate transformation cluster, you lean on the warehouse's own elastic compute, and ELT has become the default shape for new pipelines as a result. ETL hasn't disappeared — it still shows up where compliance rules require certain data to be scrubbed or masked before it's ever persisted anywhere, warehouse included — but for most greenlight-today pipelines, "load first, transform in SQL" is the path of least resistance.

## The star schema — a model built for aggregation, not updates

An operational schema is usually **normalized**: each fact lives in exactly one place, so an update only ever has to touch one row, and you never get two tables disagreeing about a customer's address. That's the right trade-off for a system optimized around writes.

A warehouse inverts that trade-off, because its job is reads that aggregate across millions of rows, not single-row writes. The classic warehouse model is a **star schema**: one large **fact table** holding the measurable events you care about (one row per order line, one row per page view, one row per transaction) — each referencing a small set of **dimension tables** that describe the who/what/when/where of that event (customer, product, date, region). Drawn out, the fact table sits at the center with dimension tables radiating outward like points on a star. A query like "total revenue by region, by month" becomes a handful of joins from the fact table to two or three small dimension tables and a `GROUP BY` — fast, even over billions of rows — instead of the many-table join chain a fully normalized operational schema would require for the same question.

## Where an architect's job actually starts

None of this is about picking a tool — it's about recognizing, early, that "store the data" and "answer questions about the data at scale" are two different problems with two different access patterns, and architecting for the one your system actually needs usually means budgeting for both from the start. A system that tries to serve both out of one database usually works fine in a demo and then falls over the first time someone runs a real report during peak traffic. Deciding where that boundary sits — what's operational, what's analytical, and what pipeline connects them — is squarely an architecture decision, which is why it belongs in this roadmap rather than a database-internals one.
