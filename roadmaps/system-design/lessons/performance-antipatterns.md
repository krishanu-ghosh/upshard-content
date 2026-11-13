---
title: "Performance Antipatterns"
description: Recognizable failure patterns — busy database, chatty I/O, sync-where-async-would-do, noisy neighbor, extraneous fetching — and how to fix each.
roadmap: system-design
node: performance-antipatterns
group: reliability-and-resilience-patterns
video:
  provider: youtube
  id: "N8NWDHgWA28"
  start: 0
  end: null
creator:
  name: GOTO Conferences
  channel_url: https://www.youtube.com/@GOTO-
duration_seconds: 2418
level: intermediate
prerequisites:
  - connection-pooling-and-read-write-splitting
takeaways:
  - A performance antipattern is a design choice that works fine at low load and degrades badly as load increases — the bug isn't visible until traffic finds it.
  - The busy database antipattern pushes logic (computation, formatting, business rules) into the database that application servers could do instead, turning the database into the bottleneck everything else is waiting on.
  - Chatty I/O makes many small round trips to accomplish one logical operation, where each round trip's network latency adds up even though each individual request is cheap.
  - Synchronous I/O blocks a worker thread for the duration of a slow call it isn't using the CPU for, capping overall throughput at (thread count) / (call duration) regardless of how fast the CPU itself is.
  - The noisy neighbor problem is one tenant or workload consuming a disproportionate share of a shared resource, degrading performance for every other tenant sharing it.
  - Extraneous fetching retrieves more data than a request actually needs — whole rows for two columns, whole result sets for one page — turning an I/O cost into a bigger I/O cost than the task required.
check_questions:
  - q: Why does the busy database antipattern get worse specifically as a system scales, rather than being a problem from day one?
    a: At low traffic a database doing extra computation work has spare capacity to absorb it, but as request volume grows the database is usually the one component that's hardest to horizontally scale, so logic that could have run on easily-scaled application servers instead competes for capacity on the one resource that can't easily grow with demand.
  - q: An API endpoint fetches a shopping cart by making five separate calls — one per line item — to a pricing service, each taking 20ms. What antipattern is this, and what's the general fix?
    a: This is chatty I/O; the fix is batching the five lookups into a single call that accepts a list of item IDs and returns all five prices at once, paying the network round-trip cost one time instead of five.
  - q: A web server uses one thread per incoming request, and each request makes a blocking call to a slow downstream service that takes 2 seconds to respond. With 100 threads available, what's the server's maximum throughput, and why?
    a: Roughly 50 requests per second (100 threads ÷ 2 seconds each), because every thread sits idle, blocked on I/O, for the full 2 seconds of each call — the CPU is barely used, but throughput is still capped by thread count, which is exactly the problem asynchronous I/O (not blocking a thread while waiting) is designed to avoid.
---

## Antipatterns only show up under load

A performance antipattern is a specific, recognizable shape of design mistake: something that works perfectly well in development, in a demo, and often in production for months — and then degrades, sometimes catastrophically, once traffic or data volume crosses some threshold. These patterns recur across unrelated codebases because they usually come from a reasonable-sounding local decision ("just query the database for this" / "just call the service per item") that doesn't account for what happens when that decision runs thousands of times a second. Knowing the common shapes means being able to spot them in a design review instead of waiting for them to show up as an incident.

## Busy database

A database is good at storage, indexing, and transactional guarantees. It's generally not the cheapest place to run arbitrary computation, string formatting, or business logic — yet it's an easy trap to fall into, because it's convenient to write a stored procedure or a complex query that does the formatting/calculation right there next to the data. The problem is that a database is usually the hardest tier in the whole system to scale horizontally; application servers can be added trivially behind a load balancer, but a primary database often can't be, at least not for writes. Pushing avoidable work onto the one tier that scales worst means that tier becomes the ceiling on the whole system's throughput well before anything else does. The fix is almost always to move computation that doesn't need to happen *in* the data store out to the application layer, and keep the database doing what only it can do: storing and retrieving data correctly and efficiently.

## Chatty I/O

Chatty I/O is making many small, separate network calls to accomplish one logical unit of work, where the problem isn't any single call's cost but the sum of their round-trip latencies. Fetching a shopping cart's five line items by calling a pricing service five separate times, once per item, is the canonical example: even if each call takes a cheap 20ms, the sequential cost compounds to 100ms before the cart can even be assembled — and that's before accounting for the fact that each call has fixed overhead (connection setup, serialization) on top of its processing time. The fix is batching: a single call that accepts a list of item IDs and returns all five prices pays the network round-trip cost once. The general principle — fewer, larger calls beat more, smaller ones, for anything that crosses a network — applies equally to database queries (one query with a `WHERE id IN (...)` instead of N queries in a loop) and to internal service calls.

## Synchronous I/O where async would do

A thread blocked on a network call isn't doing useful work — it's just occupying a slot while it waits. In a thread-per-request model, a server with 100 worker threads and a downstream call that takes 2 seconds can serve at most 50 requests per second (100 ÷ 2), no matter how fast the server's CPU is, because every thread spends essentially all its time idle, waiting, rather than computing. This is the synchronous I/O antipattern: using a scarce resource (a thread) to represent "waiting for something slow" when waiting doesn't require that resource at all.

The fix is asynchronous I/O — issuing the call and freeing the thread to do other work (serve a different request) while the response is pending, then resuming when it arrives. This is the entire reason event-loop architectures (Node.js, async frameworks in most modern languages) and reactive programming models exist: they let one thread juggle many concurrent in-flight I/O operations instead of needing one thread held hostage per operation. The tradeoff is code complexity — async code is harder to reason about than a straight top-to-bottom blocking call — which is why synchronous I/O remains the right default for genuinely low-concurrency situations and only becomes an antipattern once concurrent load is high enough that thread count becomes the bottleneck.

## Noisy neighbor

When multiple tenants, workloads, or requests share a finite resource — CPU on a multi-tenant host, IOPS on a shared disk, connections in a shared pool — one consumer using a disproportionate share degrades performance for everyone else on that resource, even if they have nothing to do with each other logically. A single tenant running an expensive batch report on a multi-tenant database can slow down every other tenant's ordinary queries, purely because they're competing for the same disk I/O and buffer cache. The fix is some form of isolation or fairness: per-tenant resource quotas, separate connection pools or compute per tenant (the same idea as a bulkhead, applied to multi-tenancy specifically), or throttling the heavy consumer so its usage can't exceed its fair share.

## Extraneous fetching

Extraneous fetching is retrieving more data than the task actually needs: `SELECT *` when only two columns are used, loading an entire object graph when only one field is displayed, or pulling back a full result set when only one page of it will ever be shown to a user. Each case turns what should be a small, cheap I/O operation into a larger one — more bytes over the network, more memory held, more deserialization work — for no benefit, since the extra data is discarded immediately after fetching. The fix is almost mechanical once spotted: select only the columns needed, paginate instead of fetching everything, and project API responses down to the fields a client actually consumes rather than returning the full underlying record by default.

## The common thread

Every one of these antipatterns is a mismatch between where a cost is paid and where capacity actually exists to absorb it — work landing on the database instead of the app tier, latency landing on a blocked thread instead of being overlapped with other work, I/O volume landing on the network instead of being trimmed to what's needed. Spotting them is mostly a matter of asking, for any given piece of work: is this happening on the component best equipped to absorb it, or just the one that was easiest to write?
