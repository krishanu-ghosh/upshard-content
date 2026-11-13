---
title: "Load Balancing Algorithms"
description: How round robin, weighted round robin, least connections, consistent hashing, and power-of-two-choices pick which server gets each request.
roadmap: system-design
node: load-balancing-algorithms
group: load-balancing-and-service-discovery
video:
  provider: youtube
  id: "dBmxNsS3BGE"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 318
level: intermediate
prerequisites:
  - dns-and-request-routing-at-scale
takeaways:
  - Round robin assumes every server has equal capacity and every request costs the same amount of work, and it has no way to detect when either assumption is false.
  - Weighted round robin corrects for known, static capacity differences between servers, but it cannot react to runtime conditions like a cold cache or a slow neighbor.
  - Least connections routes each request to whichever backend currently has the fewest active connections, which reacts to real-time load but requires the balancer to track live per-backend state.
  - Consistent hashing maps servers and keys onto the same hash ring so that adding or removing one server only reassigns the keys nearest to it, instead of reshuffling the entire keyspace.
  - Power-of-two-choices samples just two random backends per request and picks the less-loaded one, capturing most of least-connections' benefit at a fraction of the coordination cost.
  - The right algorithm depends on whether requests have unequal cost, whether any backend holds per-key state, and how many independent balancers are deciding at once.
check_questions:
  - q: Why does round robin perform badly when one server in the pool has just restarted and is rebuilding a local cache?
    a: Round robin keeps sending that server its normal share of requests on a fixed schedule regardless of how slow it currently is, because the algorithm only counts requests and has no signal for a backend's actual processing capacity at that moment.
  - q: What specifically goes wrong if you load-balance a cache with plain modulo hashing (hash(key) % N) and then add one more server?
    a: Changing N changes the result of the modulo operation for nearly every key, so almost every key gets reassigned to a different server all at once, which for a cache means most of it is treated as a miss and has to be rebuilt.
  - q: Why does power-of-two-choices avoid the pile-up problem that plain least-connections can suffer when several independent load balancers run at once?
    a: Because each balancer picks its two candidates at random before comparing their load, the choices made by different balancers are decorrelated, so they don't all converge on sending traffic to the same backend the way they can when every balancer deterministically picks the single "least loaded" server.
related_formulas: []
---

## Why the simplest algorithm breaks first

Round robin load balancing sends request 1 to server A, request 2 to server B,
request 3 to server C, and then wraps back around. It needs no state beyond a
counter, and under one condition it works beautifully: every server is
identical in capacity, every request costs about the same amount of work, and
every connection is short-lived. Real traffic violates all three.

Picture a fleet of three app servers, one of which was just restarted and
spent the last few minutes building a local cache from cold storage. Round
robin keeps sending it a third of the traffic anyway, oblivious to the fact
that its response times are now five times slower. Or picture an e-commerce
endpoint where most requests are cheap read-only product lookups but one in
every hundred is a multi-second checkout that holds a database connection
open. Round robin has no concept of "this request is expensive" — it only
counts requests, not the work behind them. Slow requests pile up on whichever
server happens to receive them, and because the algorithm keeps feeding that
server more traffic on schedule, the pile grows instead of draining.

This is the core failure mode to understand before touching anything
fancier: round robin assumes homogeneity it cannot verify. Every other
algorithm here is a different way of injecting real information — capacity,
current load, or request identity — into the choice of which server handles
the next request.

## Weighted round robin: fixing capacity, not load

Weighted round robin keeps round robin's ordering but gives each server an
integer weight proportional to its capacity. A server rated at weight 3 gets
three requests for every one a weight-1 server gets. This is the right fix
when the imbalance is static and known ahead of time — a mixed fleet where
some boxes have twice the CPU of others, for example. It does nothing,
however, for imbalance that shows up at runtime: a cold cache, a GC pause, a
noisy neighbor on shared hardware. Weights are set once, or recalculated on a
slow schedule; they don't react to what's happening right now.

## Least connections: reacting to load in real time

Least connections flips the question from "whose turn is it" to "who's least
busy right now." The load balancer tracks how many connections (or in-flight
requests) each backend currently has open, and routes the next request to
whichever server has the fewest. This handles the slow-restart and
expensive-checkout cases above directly: a server with five slow requests
already queued looks busier than one with zero, so it stops receiving new
work until it catches up.

The tradeoff is state and visibility. The balancer has to track live
connection counts per backend, which is trivial for a single load balancer
process but gets harder once several load balancer instances run in parallel
with no shared view of each other's counts — each one thinks it's the only
one choosing, and all of them can pile onto the same "apparently idle"
backend simultaneously. A refinement, least response time, factors in recent
latency as well as connection count, so a backend that's technically idle but
has been answering slowly doesn't get flooded either.

## Consistent hashing: stability under a changing pool

Round robin and least connections both assume you don't care which specific
server handles a given request, only that load is spread evenly. That
assumption breaks for anything with per-key state: a cache where each node
holds a different slice of the keyspace, or a session store where a client's
data lives on one particular shard. Here you need the same key to land on the
same server every time — but you still want to add or remove servers without
reshuffling everything.

Plain modulo hashing (`hash(key) % N`) fails this badly: change `N` by adding
one server, and nearly every key's assigned server changes, which for a cache
means a near-total cache wipe. Consistent hashing fixes this by mapping both
servers and keys onto the same ring — imagine hash values arranged in a
circle. A key is assigned to the first server found walking clockwise from
its position on the ring. Adding a server only steals the keys between it and
the next server clockwise; everyone else's assignment is untouched. In
practice, each physical server is placed at multiple points on the ring
(virtual nodes) so that the keys it picks up or gives away are spread evenly
rather than dumped on one unlucky neighbor.

## Power-of-two-choices: almost as good, far cheaper

Least connections needs to know every backend's current load, which doesn't
scale cleanly to very large pools or multiple independent balancers.
Power-of-two-choices is the practical middle ground: for each request, pick
two backends at random, check only those two for load, and send the request
to the less-loaded of the pair.

The surprising result, backed by queueing theory, is that checking just two
random options gets you most of the benefit of checking all of them — the
probability of a bad pick drops sharply compared to picking one server at
random, while the cost of the check stays constant regardless of pool size.
This is why power-of-two-choices shows up inside large-scale systems like
Envoy's default routing policy: it avoids the "thundering herd onto the same
backend" failure that naive least-connections suffers when multiple
independent balancers converge on the same answer, because the randomness
decorrelates their choices.

## Picking an algorithm for a real system

In practice these compose rather than compete: a DNS or edge layer might do
coarse geographic routing, an L4/L7 load balancer in front of a stateless API
tier runs least connections or power-of-two-choices, and a caching or
sharding layer behind it uses consistent hashing to keep key ownership
stable. The question to ask for any tier is simple: do requests have
meaningfully different costs, does any backend hold state tied to specific
keys, and how many independent balancers are making this decision at once?
The answer tells you which algorithm actually matches the failure modes
you'll hit.
