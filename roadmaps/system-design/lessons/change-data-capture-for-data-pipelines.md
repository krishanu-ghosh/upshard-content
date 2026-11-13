---
title: "Change Data Capture for Data Pipelines"
description: Streaming a database's write-ahead log to other systems without dual writes — and the transactional outbox pattern that backs it.
roadmap: system-design
node: change-data-capture-for-data-pipelines
group: async-systems-and-messaging
video:
  provider: youtube
  id: "ARWNnnDGPag"
  start: 0
  end: null
creator:
  name: CodeOpinion
  channel_url: https://www.youtube.com/@CodeOpinion
duration_seconds: 386
level: advanced
prerequisites:
  - pub-sub-and-event-driven-architecture
  - acid-transactions-and-the-cap-theorem
takeaways:
  - A dual write — writing to your database and separately publishing an event or writing to a search index in the same request — can fail halfway through, leaving the database and the other system permanently disagreeing about what happened.
  - Change data capture (CDC) avoids dual writes by reading the database's own write-ahead log or binlog — the internal record it already keeps of every committed change — and turning each entry into an event for other systems to consume.
  - Because CDC reads a log the database produces as a side effect of committing a transaction, it only ever sees changes that actually committed, which is what makes it safe to rely on instead of a second, separate write that might not happen.
  - The transactional outbox pattern writes the "event to publish" into an outbox table in the very same local transaction as the business data change, so a CDC process (or a simple poller) can reliably pick it up afterward — turning "write data and publish an event" into one atomic local write.
  - CDC is commonly used to keep a search index, a cache, or a data warehouse in sync with a primary database's writes, without ever modifying the application code that performs those writes.
  - "CDC and event sourcing solve related but different problems: CDC captures changes to data that already has one authoritative, mutable home as a side effect; event sourcing makes the event log itself that authoritative home, with no separate mutable row underneath it at all."
check_questions:
  - q: An application writes an order to its database, then makes a separate call to push that order into a search index. The search-index call times out. What state is the system left in, and why is this the core problem CDC solves?
    a: The order now exists in the database but not in the search index, and there's no automatic way to detect or fix that drift because the two writes were independent operations with no shared transaction; CDC solves this by never performing a second write at all — it reads the change from the database's own committed log, so the search index only ever reflects writes that actually happened and can't drift out of sync for this reason.
  - q: Why does reading a database's write-ahead log give CDC a stronger guarantee than having the application code publish an event right after it commits a write?
    a: The write-ahead log only contains changes that the database actually committed, and a CDC tool reading it will eventually pick up every single one, whereas application code publishing an event after a commit is a second, separate action that can fail, crash, or be skipped independently of whether the database write succeeded — the log-based approach ties the event's existence directly to the fact that the data change really happened.
  - q: How does the transactional outbox pattern let a service "publish an event" without a separate, independently-failing publish step?
    a: It writes a row describing the event into an outbox table as part of the exact same local database transaction that changes the business data, so either both the data change and the outbox row commit together or neither does; a separate process (often CDC reading the log, or a simple poller) then reads committed outbox rows and actually publishes them, so the risky "publish to the outside world" step happens after the atomic part is already safely done.
deeper: []
related_formulas: []
---

## The dual-write problem

Say a service needs to do two things when an order is placed: write the order to its database, and tell a search index about it so the order shows up in search results. The obvious way to do this is to write to the database, then separately call the search index's API. That's a **dual write** — two independent operations that both need to succeed for the system to stay consistent — and it has a structural weakness: there's no way to make both happen atomically across two different systems. The database write can succeed while the search-index call times out, crashes, or simply never runs, and now the two systems permanently disagree about what exists, with nothing watching for that drift or correcting it.

This isn't a rare edge case — it's the default outcome of doing two separate writes over a network, given enough volume and enough time. **Change data capture** exists specifically to avoid it, by removing the second, independent write altogether.

## Reading the log instead of writing twice

Every relational database already keeps an internal, append-only record of every change it commits — PostgreSQL calls this the write-ahead log (WAL), MySQL calls it the binlog. The database writes to this log *as part of* committing a transaction; it's not optional, and it's not a second write your application has to remember to make. **Change data capture (CDC)** works by having a separate process tail that log and turn each entry into an event: "this row was inserted," "this row was updated from X to Y," "this row was deleted."

```
App writes order  →  DB commits  →  WAL/binlog entry written (automatic, same commit)
                                            │
                                   CDC process tails the log
                                            │
                                   event published: "order 441 inserted"
                                            │
                              ┌─────────────┼─────────────┐
                              ▼             ▼              ▼
                        search index     cache          data warehouse
```

The application code that writes the order never changes — it doesn't know CDC exists, doesn't call any extra API, and doesn't have a second write that can fail independently. CDC only ever sees changes that actually committed, because it's reading the exact record the database itself produced as a consequence of committing them. That's the core guarantee that dual writes can't offer: there is no window where the primary write happened but the downstream event didn't, because the event's existence is tied directly to the commit, not to a separate call that might not run.

Tools like Debezium are the common way to implement this in practice — they connect to a database's log, decode the vendor-specific log format into a normalized stream of change events, and typically publish those events onto a broker like Kafka for anything downstream to consume (the pub/sub fan-out from the previous lesson is exactly what lets a search index, a cache invalidator, and a warehouse loader all consume the same stream of change events independently).

## The transactional outbox pattern

CDC solves the problem for changes to data the application already stores in its database. But sometimes a service wants to publish a genuinely new *event* — not just "this row changed," but something with its own shape, like "OrderPlaced" with a specific payload — and that still runs into the same dual-write risk if it's done as a second, separate publish step after the database commit.

The **transactional outbox pattern** fixes this by making the event-to-publish part of the same local transaction as the data change. The service writes its business data *and* a row describing the event into an `outbox` table, in one transaction:

```sql
BEGIN;
  INSERT INTO orders (id, customer_id, total) VALUES (441, 9, 58.00);
  INSERT INTO outbox (event_type, payload) VALUES ('OrderPlaced', '{"order_id": 441, ...}');
COMMIT;
```

Either both rows commit or neither does — there's no window where the order exists but the outbox event doesn't, because they're the same local database transaction, which is a guarantee a single database already gives you for free. A separate process then reads committed outbox rows and actually publishes them to the broker — and that reader is very often a CDC process, tailing the log for changes to the outbox table specifically, which turns "reliably publish an event after a local write" into the same log-tailing mechanism CDC already uses for everything else.

## CDC is not event sourcing

It's worth being precise about a distinction that's easy to blur: CDC captures changes to data that already has one authoritative, mutable home — the row in the orders table — as a byproduct of that row changing. **Event sourcing** is a different design entirely, covered in its own lesson, where the event log itself *is* the authoritative source of truth and there's no separate mutable row underneath it at all; the "current state" is derived by replaying events, not stored independently of them. CDC assumes a conventional mutable database and taps its log as a side channel; event sourcing has no conventional mutable database to tap in the first place.
