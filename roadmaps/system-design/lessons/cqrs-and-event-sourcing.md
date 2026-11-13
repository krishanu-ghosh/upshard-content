---
title: "CQRS and Event Sourcing"
description: Why separating reads from writes (CQRS) and treating an event log as the source of truth (event sourcing) are a natural pairing — and their real costs.
roadmap: system-design
node: cqrs-and-event-sourcing
group: architecture-patterns-at-scale
video:
  provider: youtube
  id: "5aznkIEvkKc"
  start: 0
  end: null
creator:
  name: CodeOpinion
  channel_url: https://www.youtube.com/@CodeOpinion
duration_seconds: 875
level: advanced
prerequisites:
  - monolith-vs-microservices
takeaways:
  - CQRS splits the model used to write data (commands) from the model used to read it (queries), instead of using one model and one schema for both.
  - Event sourcing stores every change as an immutable event in an append-only log, and treats that log — not a mutable row — as the source of truth.
  - A "current state" row or table is a projection, derived by replaying events; you can delete it and rebuild it from the log, which you cannot do with a normal mutable database.
  - CQRS and event sourcing are often used together but are independent choices — you can do CQRS with a normal mutable write database, and you can event-source a system with a single unified read/write model.
  - The event log makes "what happened and why" a first-class, permanently queryable fact, which is why it's a natural fit for audit trails, analytics, and rebuilding read models after a bug.
  - The cost is real complexity — eventual consistency between write and read sides, and versioning events as the schema evolves — so this pairing is a fit for a specific kind of problem, not a default architecture.
check_questions:
  - q: What does CQRS actually separate, and what stays the same underneath?
    a: It separates the model and code path used to change data (a command, validated against business rules) from the model and code path used to read data (a query against a model shaped for display) — the underlying facts about the system don't change, only how you write to versus read from it.
  - q: In an event-sourced system, what is a "projection," and why can you safely throw one away?
    a: A projection is a read-optimized view — like a current-balance table — built by replaying the event log and folding each event into the view; you can safely delete and rebuild it because the event log, not the projection, is the source of truth.
  - q: Why does an event-sourced system typically need the saga pattern for any operation that spans multiple aggregates or services?
    a: Because there's no cross-aggregate ACID transaction to lean on — each aggregate's events commit independently, so coordinating a multi-step operation across them needs an explicit sequence of steps with compensating actions if one fails, which is exactly what the saga pattern provides (see `distributed-transactions-2pc-and-sagas`).
related_formulas: []
---

## The problem CQRS solves

Most systems default to one model: a `users` table, an `orders` table, with one ORM class per table, used identically for both writing a new order and rendering an order history page. That's fine until the read and write sides pull in genuinely different directions. Writing an order needs strict validation, business rules, and a normalized schema that avoids duplicate data. Reading a dashboard needs a flat, pre-joined, maybe pre-aggregated shape that's fast to query and might not need a perfectly up-to-the-millisecond number.

**Command Query Responsibility Segregation (CQRS)** takes that seriously: the write side exposes commands — `PlaceOrder`, `CancelOrder` — that go through validation and business logic and change the authoritative state. The read side exposes queries against a separate model, often a separate denormalized store, built specifically to answer the questions the UI or reports actually ask. The two models can even live in different databases: a normalized SQL store for writes, a document store or search index for reads.

The segregation is worth it when the read and write shapes have really diverged, and especially when read volume vastly outstrips write volume (a product catalog gets read millions of times for every write) or when different parts of the system need very different scaling — you can scale read replicas independently of the write path. It's not worth it for a simple CRUD screen where one model serves both sides fine; adding a second model and a synchronization path there is pure overhead.

## The problem event sourcing solves

A normal database keeps a `balance` column and overwrites it on every transaction — the history of *how* it got there is gone the moment the next write lands. **Event sourcing** inverts that: instead of storing current state, you store the sequence of events that produced it — `AccountOpened`, `FundsDeposited`, `FundsWithdrawn` — as an immutable, append-only log. Current state is never stored directly; it's computed by replaying the events for that entity from the beginning (or from the latest snapshot, for efficiency) and folding each one into a result.

This has consequences that a normal mutable table doesn't give you for free:

- **A full audit trail is automatic.** You don't need a separate audit_log table bolted on — the event log *is* the history, in order, with nothing lost.
- **You can answer questions you didn't anticipate.** If a bug produced wrong balances for three months, you can replay the real events through a corrected projection and get the right numbers back, because the facts (the events) were never wrong — only the derived view was.
- **You can build new views after the fact.** Need a new dashboard that didn't exist a year ago? Replay the whole event log through a new projection and you have its full history, not just data from the day you turned the dashboard on.

The cost is real: events are forever, so you need a plan for changing their shape over time (event versioning and upcasting), replaying a long log can be slow without snapshots, and most teams end up needing a dedicated event store (like EventStoreDB) or a well-disciplined append-only table to make this practical at scale.

## Why they pair naturally

CQRS and event sourcing solve adjacent problems, which is why they're so often used together. In an event-sourced system, the write side's job is simple and uniform: validate a command, append the resulting event(s) to the log. But that log is a terrible shape to query directly — "give me this customer's current balance" would mean replaying their entire history on every read. So you build **projections**: consumers that subscribe to the event stream and continuously fold new events into a read-optimized table or document, which is exactly the read model CQRS asked for. The command side appends events; the query side reads projections that are kept up to date by consuming those same events.

That consumption is asynchronous almost by necessity — the write succeeds the moment the event is durably appended, and the projection catches up shortly after. That's an explicit trade: the read side is *eventually* consistent with the write side, usually by milliseconds, occasionally by more under load. A UI that writes a value and immediately re-reads it may briefly see the old projection, which is a real design constraint, not a bug to "fix away."

## Related, but different: don't re-derive these here

Two adjacent patterns come up constantly alongside CQRS and event sourcing, and are worth knowing by name without re-deriving them in this lesson:

- **Change data capture (CDC)** is a different mechanism for a similar-sounding goal — streaming a *relational database's* row-level changes out to other systems, usually without the application ever writing explicit domain events. It's the right tool when you're integrating around an existing mutable database rather than event-sourcing a new one; see `change-data-capture-for-data-pipelines`.
- **Sagas** handle the case where a single business operation has to coordinate changes across multiple aggregates or services with no shared transaction to lean on — exactly the situation an event-sourced system is in by default, since each aggregate's own event stream is the only thing that commits atomically. The saga pattern, orchestration versus choreography, and compensating transactions get their full treatment in `distributed-transactions-2pc-and-sagas`.

Reach for CQRS and event sourcing when the write/read shapes have genuinely diverged, when a durable audit trail is valuable in its own right, or when you expect to need new read models later that today's schema can't anticipate — not as a default way to build a service.
