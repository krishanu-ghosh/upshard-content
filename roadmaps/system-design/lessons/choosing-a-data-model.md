---
title: "Choosing a Data Model: Key-Value, Document, Column-Family, and Graph"
description: NoSQL isn't one thing — key-value, document, column-family, and graph stores each suit a different access pattern.
roadmap: system-design
node: choosing-a-data-model
group: data-modeling-and-storage
video:
  provider: youtube
  id: "mMdZEAKtntA"
  start: 0
  end: null
creator:
  name: CodeLucky
  channel_url: https://www.youtube.com/@thecodelucky
duration_seconds: 265
level: intermediate
prerequisites:
  - sql-vs-nosql-and-data-modeling
takeaways:
  - A key-value store treats the value as an opaque blob addressed only by its key, which makes it extremely fast but means you can only ever look things up by that one key.
  - A document store adds internal structure to the value — usually JSON — so you can query and index on fields inside it, at the cost of being slower than a pure key-value store for simple lookups.
  - A column-family store groups related columns together on disk and is built to scan a huge number of rows while touching only a few columns, which is why it suits time-series and write-heavy telemetry data.
  - A graph store makes the relationship between two records a first-class, directly-traversable thing, instead of something you reconstruct with a join at query time.
  - The right model follows from your dominant access pattern — "how will this data actually be read" — not from which database is trendiest.
  - These four families aren't mutually exclusive inside one system; large applications commonly use more than one, each for the subset of data it actually fits.
check_questions:
  - q: Why can't you efficiently query a key-value store by anything other than its key?
    a: A key-value store doesn't look inside the value at all — it stores it as an undifferentiated blob and only indexes the key, so there's no internal structure for the database to search without reading and deserializing every value yourself.
  - q: Why is a column-family store a good fit for time-series sensor data but a poor fit for an e-commerce order with twenty related attributes you always read together?
    a: Column-family stores are optimized for scanning one or two columns across millions of rows (like reading just the "temperature" column for a sensor over a time range), not for fetching every attribute of a single record at once, which is what a wide, all-attributes-together read pattern like an order actually needs.
  - q: What does a graph database do differently from a relational database when you ask "find all of this person's friends-of-friends"?
    a: A relational database answers that by joining the friendship table against itself, which gets slower as the join depth grows; a graph database stores the friendship edges directly on each node, so walking two hops out is a constant-time traversal from node to node rather than a join that re-scans a table.
deeper: []
related_formulas: []
---

## Four families, one underlying question

The previous lesson covered *why* you might leave a relational database behind — flexible schemas, horizontal scale, avoiding expensive joins at scale. But "NoSQL" was never one technology. It's an umbrella over at least four genuinely different storage models, and picking the wrong one for your access pattern causes exactly the kind of pain people blame on "NoSQL" generally. The question that actually matters for each one is the same: **how is this data going to be read?**

## Key-value: the simplest possible contract

A key-value store is the simplest model there is: give it a key, get back a value. The database doesn't know or care what's inside that value — it's an opaque blob of bytes as far as the storage engine is concerned. Redis and DynamoDB (in its simplest mode) are the canonical examples.

```
SET session:8f3a2  {"user_id": 441, "expires": 1719600000}
GET session:8f3a2  → {"user_id": 441, "expires": 1719600000}
```

That simplicity is the whole point. Because there's no schema to validate and no secondary structure to maintain, a lookup by key is about as fast as a database can go, which is exactly why key-value stores dominate session storage, caching layers, and shopping carts — things you always fetch by one known ID and never need to search by content. The tradeoff is equally absolute: if you ever need "find all sessions expiring in the next hour," a pure key-value store can't do that without you scanning and deserializing every value yourself.

## Document: structure without a fixed schema

A document store, like MongoDB or Couchbase, keeps the key-value idea but opens up the value. The value is a structured document — typically JSON — and the database can see inside it: index specific fields, query on nested attributes, and let different documents in the same collection have different shapes.

```json
{
  "_id": "user_441",
  "name": "Priya",
  "addresses": [
    { "type": "home", "city": "Austin" },
    { "type": "work", "city": "Denver" }
  ]
}
```

This fits data that's naturally hierarchical and doesn't need to be split across tables — a user profile with a variable number of addresses, a product catalog where different categories have different attributes. The cost is that document stores are typically weaker at the kind of multi-record, cross-collection joins and aggregations a relational database does natively; you either denormalize (duplicate data across documents) or do the join in application code.

## Column-family: wide rows built for scanning

A column-family store — Cassandra and HBase are the standard examples — organizes data differently again: rows can have a huge, sparse, and even per-row-varying set of columns, and those columns are physically grouped on disk so that reading *one column across millions of rows* is cheap, while reading *every column of one row* is comparatively more work.

```
Row key: sensor_8842
  2024-01-01T00:00 → { temp: 21.3 }
  2024-01-01T00:05 → { temp: 21.4 }
  2024-01-01T00:10 → { temp: 21.1 }
```

That access pattern — append a huge, continuous stream of narrow writes, then later scan a time range for one or two fields — is exactly what IoT telemetry, metrics, and event logging look like. It's also why column-family stores are built for very high write throughput: writes are mostly sequential appends, not random-access updates to a complex row.

## Graph: relationships as first-class data

A graph store — Neo4j is the best-known — flips the usual priority. In a relational database, a relationship between two rows is implicit: it's a foreign key, and finding connected records means a join. In a graph database, the relationship itself is a stored, directly-traversable edge with its own properties, sitting right next to the nodes it connects.

```
(Alice)-[:FOLLOWS]->(Bob)-[:FOLLOWS]->(Carol)
```

Finding "everyone Alice follows" is one hop along a stored edge. Finding "friends of friends," three hops deep, is still cheap — you're walking pointers, not re-joining a table against itself at increasing depth. This is why social graphs, fraud-ring detection, and recommendation engines ("people who bought X also bought Y, which was also bought by...") reach for a graph store: the query that's natural to ask — "what's connected to this, and what's connected to that" — is the query the storage engine was built to answer directly.

## Picking the right one

None of these four is a strictly better NoSQL — they're different bets on what you'll ask the data later. A session cache wants key-value's speed. A product catalog with irregular attributes wants a document store's flexible structure. A fleet of sensors emitting a constant stream of narrow writes wants a column-family store's scan-optimized layout. A social network or a fraud graph wants a graph store's native traversal. Many real systems end up using more than one of these side by side, each holding the slice of data whose access pattern it actually fits — which is also why "choosing a data model" is a decision made per dataset, not once for the whole system.
