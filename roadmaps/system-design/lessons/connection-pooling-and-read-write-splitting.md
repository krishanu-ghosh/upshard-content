---
title: "Connection Pooling and Read/Write Splitting"
description: Why opening a connection per request doesn't scale, how pooling fixes it, and the staleness tradeoff of routing reads to replicas.
roadmap: system-design
node: connection-pooling-and-read-write-splitting
group: scaling-data-and-services
video:
  provider: youtube
  id: "jDk95_C5I1s"
  start: 0
  end: null
creator:
  name: Coding Coach
  channel_url: https://www.youtube.com/@CodingCoach
duration_seconds: 2037
level: intermediate
prerequisites:
  - replication-sharding-and-federation
takeaways:
  - Opening a database connection is expensive — a TCP handshake, then authentication, then session setup — and doing that fresh for every single request wastes most of the request's time on setup rather than on the query.
  - A connection pool keeps a set of already-open connections ready to hand out, so a request borrows one, uses it, and returns it instead of paying the open/close cost every time.
  - A pool has to be sized deliberately — too small and requests queue up waiting for a free connection; too large and you can exhaust the database server's own connection limit, which is usually far lower than your application's request concurrency.
  - Read/write splitting sends writes to the primary and spreads reads across one or more replicas, which is the same replication setup from the previous lesson, now used specifically to multiply read capacity.
  - Because replicas apply writes with some lag, a read routed to a replica can return data that's milliseconds-to-seconds out of date — the same replication-lag tradeoff already covered, now showing up as an explicit routing decision your application has to make.
  - Routing which queries go to the primary versus a replica can happen in application code, in an ORM's read/write connection settings, or transparently in a proxy like PgBouncer or ProxySQL sitting in front of the database.
check_questions:
  - q: Why is reusing a pooled connection faster than opening a fresh one for every request, even though both eventually run the same SQL query?
    a: Opening a connection requires a TCP handshake, authentication, and session negotiation before a single query can run, and all of that setup cost is paid again on every new connection; a pooled connection has already paid that cost once, so borrowing it skips straight to running the query.
  - q: What goes wrong if a connection pool's maximum size is set higher than the database server can actually handle?
    a: Each open connection consumes memory and a slot on the database server itself, so a pool sized past the server's real connection limit can exhaust the server's capacity and start rejecting or killing connections — the pool protects the application's concurrency, but it has to stay within what the database can actually sustain.
  - q: A user updates their email address and the page immediately re-fetches their profile to confirm the change — why might the confirmation show the old email?
    a: If the write went to the primary but the confirming read was routed to a replica, replication lag means the replica may not have applied that write yet, so it still returns the pre-update row until it catches up — exactly the staleness tradeoff that comes with splitting reads onto replicas.
deeper: []
related_formulas: []
---

## The cost hiding inside "just open a connection"

It's easy to think of a database connection as free — call a function, get a connection, run a query. Underneath, opening one actually means a full TCP handshake with the database server, then an authentication exchange, then the server allocating memory and setting up session state for that specific connection. For a single request that work might take a few milliseconds; the query itself might take one millisecond. Do that setup on every incoming request, under real traffic, and most of your database time is spent connecting, not querying.

It gets worse under load. Each process or thread handling a request that opens its own connection means your database server's open-connection count tracks your application's request concurrency directly. Most database servers cap how many connections they'll hold open at once — PostgreSQL defaults to around 100 — because each one reserves memory whether or not it's doing anything. A traffic spike that briefly needs a few hundred concurrent requests can blow straight through that limit and start getting connections refused, which is a database outage caused entirely by connection overhead, not by query load.

## Connection pooling: pay the setup cost once

A **connection pool** breaks the link between "one request" and "one freshly opened connection." The pool opens a fixed number of connections up front and keeps them alive. When a request needs the database, it borrows an already-open connection from the pool, runs its query, and returns the connection — it doesn't close it.

```
Pool (size 20): [conn1] [conn2] [conn3] ... [conn20]

Request A → borrow conn3  → query → return conn3
Request B → borrow conn7  → query → return conn7
Request C → borrow conn3  → query → return conn3   (reused, no reopen)
```

This changes the shape of the problem entirely. Instead of the database seeing a connection count that scales with request volume, it sees a steady, bounded number of connections — the pool size — no matter how much traffic the application handles, because requests are queuing for a slot in the pool rather than each opening their own. A request that can't immediately get a connection waits briefly in the pool's queue rather than the database server itself being overwhelmed.

Sizing the pool is a real decision, not a default to leave alone. Too small, and requests queue and time out waiting for a connection even though the database has spare capacity. Too large, and you risk the opposite problem from before — too many held-open connections exhausting the database's own limit, especially when several application instances each run their own pool against the same database. This is exactly why external poolers like PgBouncer exist: they sit in front of the database and multiplex many application-side connections onto a much smaller number of real database connections, letting hundreds of application workers share, say, fifty actual connections to Postgres.

## Read/write splitting: using replicas for more than failover

Once a database has read replicas — covered in the previous lesson as a way to add fault tolerance and read capacity — there's a routing decision sitting right on top of connection pooling: which pooled connection does a given query actually go to?

**Read/write splitting** answers that by type of query: every write (`INSERT`, `UPDATE`, `DELETE`) goes to the primary, since only the primary accepts writes, while reads (`SELECT`) get spread across the replicas. This multiplies effective read capacity without adding more write capacity to a single primary, which is usually fine, since most applications are read-heavy.

```
Write → always primary
Read  → round-robin across replica-1, replica-2, replica-3
```

This routing can live in a few different places: directly in application code (a `readonly=True` flag that picks a different connection pool), inside an ORM that exposes separate read/write connection settings, or transparently in a database proxy that inspects each query and routes it without the application knowing replicas exist at all.

## The tradeoff this reintroduces

Read/write splitting doesn't avoid replication lag — it makes an application actively choose to accept it, on purpose, for the reads it routes to a replica. If a user writes something and then immediately reads it back, and that read lands on a replica that hasn't yet applied the write, they'll briefly see stale data. This is the same lag discussed when replication was introduced, but now it's not a background fact about the system — it's a per-query decision: does *this particular read* need the absolute latest data (route it to the primary), or can it tolerate being a few hundred milliseconds behind (route it to a replica)? Getting that decision right, query by query, is most of what makes read/write splitting work well in practice rather than just moving the staleness problem somewhere less visible.
