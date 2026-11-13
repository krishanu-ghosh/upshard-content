---
title: "Scaling Data: Replication, Sharding, and Federation"
description: Three distinct ways to grow a database past one machine — copying it, splitting it by key, and splitting it by function.
roadmap: system-design
node: replication-sharding-and-federation
group: scaling-data-and-services
video:
  provider: youtube
  id: "_1IKwnbscQU"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 521
level: advanced
prerequisites:
  - indexes-views-and-stored-procedures
takeaways:
  - Replication copies the same data onto multiple machines, usually with one leader taking writes and followers serving reads, trading some replication lag for read throughput and fault tolerance.
  - Sharding splits a dataset horizontally across machines by a shard key, so no single machine holds the whole table — but a badly chosen key creates hot shards that get disproportionate traffic.
  - Federation splits a database by function rather than by rows — separate databases per service or domain — which limits each database's size and blast radius at the cost of making cross-domain queries harder.
  - Replication lag means a follower can briefly serve stale data right after a write hits the leader, which is why read-your-own-write guarantees need deliberate design, not an assumption.
  - These three techniques solve different bottlenecks and are commonly combined — a sharded system typically also replicates each shard for fault tolerance.
check_questions:
  - q: Why might a user who just updated their profile briefly see the old version on the next page load?
    a: If that read is served by a follower replica that hasn't yet applied the write the leader just committed, replication lag means the follower is still showing the pre-update data for a short window.
  - q: What makes a shard key "bad," concretely?
    a: A shard key that doesn't distribute load evenly — like sharding by signup date when most active users signed up recently — concentrates traffic on a small number of shards, creating a hotspot while other shards sit idle.
  - q: How does federation differ from sharding, given that both involve splitting data across multiple databases?
    a: Sharding splits one logical table's rows across machines by a key, so the table conceptually stays "one thing" split up; federation splits the database by function or service — an orders database separate from a users database — so each piece is a genuinely different dataset, not a partition of the same one.
---

## One machine runs out eventually

A single database server has a ceiling: how much data fits on its disk, how many queries it can process per second, and what happens to every client if that one machine goes down. Replication, sharding, and federation are three different answers to "what do we do once we're near that ceiling" — and they solve three different problems.

## Replication: more copies for reads and safety

**Replication** keeps multiple copies of the same data on different machines. The common pattern is leader-follower: one **leader** accepts all writes, and one or more **followers** continuously apply the same writes after the fact, serving read traffic so the leader isn't the only machine handling load.

```
          writes
Client ──────────▶ Leader
                      │  replicates writes
            ┌─────────┼─────────┐
            ▼                   ▼
        Follower A          Follower B
      (serves reads)      (serves reads)
```

This buys two things: read capacity spread across several machines, and a fallback if the leader fails (a follower can be promoted). The cost is **replication lag** — a follower applies the leader's writes a little after they happen, so a read against a follower can briefly return data that's already stale on the leader. An application that writes a value and immediately reads it back needs to either read from the leader for that specific case or design around the possibility of reading its own write late.

## Sharding: splitting the data itself

Replication copies the *whole* dataset repeatedly — it doesn't help once the dataset itself is too large or too busy for one machine to hold or serve at all. **Sharding** solves that by splitting the data horizontally: each row goes to exactly one of several shards, chosen by a **shard key**.

```
shard_id = hash(customer_id) % num_shards

Shard 0: customers with hash % 3 == 0
Shard 1: customers with hash % 3 == 1
Shard 2: customers with hash % 3 == 2
```

Each shard is a smaller, independent database holding only its slice of the rows. Picking the shard key well matters enormously: a key that spreads load evenly (like a hash of a customer id) is good; a key correlated with activity (like signup date, if most traffic comes from recently-signed-up users) creates a **hot shard** that gets hammered while the others sit nearly idle — all the operational cost of sharding with little of the benefit.

## Federation: splitting by function, not by row

**Federation** (sometimes called functional partitioning) takes a different axis entirely: instead of splitting one table's rows across machines, it splits the *database* by subject area — a `users` database, a separate `orders` database, a separate `inventory` database, each potentially on its own server. Where sharding keeps the illusion of one logical table, federation accepts that these are genuinely different datasets that happen to belong to the same product.

Federation limits each individual database's size and the blast radius of an outage — if the `inventory` database goes down, user logins on the separate `users` database keep working. The cost shows up in queries that need data from more than one domain: a report joining `orders` against `users` can no longer do that in a single SQL query across one connection, and the application has to combine the results itself (or maintain a denormalized copy for that purpose).

## Combining all three

These aren't mutually exclusive — a large system commonly federates by service, shards each service's largest tables by a well-chosen key, and replicates each shard for read capacity and fault tolerance. Each technique answers a different question: replication answers "what if a machine dies or reads outpace one machine," sharding answers "what if the data itself is too big," and federation answers "what if different parts of the system shouldn't share a blast radius at all."
