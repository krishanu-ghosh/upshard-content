---
title: "Case Study: Design a Rate Limiter"
description: A worked design for a shared, distributed rate limiter that stays low-latency and avoids becoming the bottleneck it was built to prevent.
roadmap: system-design
node: design-a-rate-limiter
group: case-studies
video:
  provider: youtube
  id: "MIJFyUPG4Z4"
  start: 0
  end: null
creator:
  name: Hello Interview
  channel_url: https://www.youtube.com/@hello_interview
duration_seconds: 3357
level: advanced
prerequisites:
  - rate-limiting-algorithms
  - distributed-caching-systems
takeaways:
  - A rate limiter protects downstream services from overload, but only if the limiter itself can absorb far more traffic than the limit it's enforcing.
  - Per-instance, in-memory rate-limiting state silently breaks correctness the moment a client's requests land on more than one server behind a load balancer.
  - A shared, low-latency store (typically an in-memory distributed cache) lets every application server check and update the same counters, which is what makes the limit actually a limit.
  - The sliding-window-counter algorithm is the common middle ground between the memory cost of sliding-window-log and the edge-burst problem of fixed-window counting.
  - Because every request now does an extra network hop to the shared counter store, that store's own availability and latency become part of your overall request-latency budget.
  - When the shared limiter store is unreachable, a system has to pick a documented failure mode — fail open (allow traffic, risk overload) or fail closed (reject traffic, risk false positives) — rather than leaving it to chance.
check_questions:
  - q: Why doesn't storing rate-limit counters in each application server's local memory work correctly behind a load balancer?
    a: A client's requests can land on different servers across a session, and each server only sees the slice of requests routed to it, so no single server ever has an accurate count of that client's total request rate, which lets clients exceed the intended limit simply by spreading requests across instances.
  - q: Why is a distributed in-memory cache the usual choice for the rate limiter's shared state, rather than a relational database?
    a: Rate-limit checks happen on nearly every request and must add minimal latency, and an in-memory store with atomic increment operations can answer "allow or deny" in sub-millisecond time, whereas a relational database's transactional overhead would make the limiter itself a latency bottleneck on the request path it's supposed to protect.
  - q: What's the tradeoff between the sliding-window-log and sliding-window-counter rate-limiting algorithms?
    a: Sliding-window-log keeps a timestamp for every individual request and is perfectly precise but uses memory proportional to the number of requests in the window, while sliding-window-counter approximates the same idea with a couple of counters and a weighted calculation, trading a small amount of precision for memory and computation that stays constant regardless of request volume.
deeper:
  - label: "Rate Limiter System Design: Token Bucket, Leaky Bucket, Scaling (ByteByteGo)"
    url: "https://www.youtube.com/watch?v=YXkOdWBwqaA"
related_formulas: []
---

## Requirements

A rate limiter's job sounds simple — reject requests once a client exceeds some quota — but "design a rate limiter" as a system-design problem is really asking: how do you make that enforcement correct and fast across a fleet of stateless servers that don't otherwise share memory?

Functional requirements:
- Enforce a limit (e.g., 100 requests/minute per API key, or per IP, or per user ID).
- Apply the limit consistently no matter which application server instance handles a given request.
- Support different limits for different clients or endpoints (a paid tier vs. a free tier, a cheap read endpoint vs. an expensive write endpoint).
- Return a clear signal to the client when they're throttled (HTTP 429, often with a `Retry-After` header).

Non-functional requirements:
- **Low added latency**: the limiter sits in front of every request, so it must add single-digit milliseconds at most, not tens.
- **High availability**: a rate limiter that goes down shouldn't necessarily take the whole service down with it.
- **Horizontal scalability**: the limiter has to handle at least as much throughput as the service it protects, often more, since it sees every request including the ones it rejects.
- **Accuracy under concurrency**: two near-simultaneous requests from the same client shouldn't both slip through a limit that should have caught the second one (a classic race condition if counter reads and writes aren't atomic).

## Capacity estimate

Say the overall service handles 500,000 requests/second at peak across its fleet, and every one of those requests needs a rate-limit check.

- The limiter's read+write throughput requirement is therefore also ~500,000 ops/second at peak — it sees 100% of traffic, not a sampled fraction.
- If each rate-limit check is a single atomic increment-and-compare against an in-memory store, and that store can do on the order of 100,000+ ops/second per node, the limiter tier alone needs on the order of 5-10 nodes just to keep up, before accounting for replication or headroom.
- Each tracked key (e.g., `user:12345:endpoint:/checkout`) is tiny — a counter and a timestamp or two, well under 100 bytes — so for 50 million active users this is a few gigabytes of state, trivially cacheable in memory, but it does mean the limiter's bottleneck is almost never storage size; it's operations-per-second and network round-trip latency.

## High-level design

Every incoming request passes through a rate-limiting check before it reaches the actual service logic — commonly implemented as middleware in a reverse proxy or API gateway layer, so application code doesn't have to think about it on every endpoint (see [reverse proxies and API gateways](/roadmaps/system-design/reverse-proxies-and-api-gateways)). That check asks a shared, low-latency data store — typically Redis or a similar in-memory distributed cache — for the client's current count in the active window, atomically increments it, and compares against the configured limit. If under the limit, the request proceeds; if over, the gateway returns 429 immediately without the request ever reaching the backend service. The shared store is itself deployed as a small, dedicated cluster, separate from any application-level caching, precisely because its availability and latency are now load-bearing for every single request in the system, not just a performance optimization.

## Key decisions

**1. Making the limiter itself distributed without making it the new bottleneck.** This is the defining tension of the whole problem. The naive version — each application server keeps its own in-memory counters — fails correctness immediately, because a load balancer spreads one client's requests across many servers, and no single server ever sees the client's true total rate (see [rate-limiting algorithms](/roadmaps/system-design/rate-limiting-algorithms) for the algorithms themselves). The fix is centralizing the counters in a shared store everyone can reach — but that store now has to sustain the same request volume as the service it protects, with lower latency than the service itself, or it defeats its own purpose. The practical answer is an in-memory distributed cache cluster, sized and scaled independently from the main application tier, using atomic operations (`INCR` plus a TTL, or a Lua script for multi-step checks) so that concurrent requests from the same client can't race past each other between the "read the count" and "write the incremented count" steps.

**2. Which algorithm, and where its cost actually lands.** Token bucket and leaky bucket smooth bursts well and are cheap to evaluate per request, but can let a client burst up to the bucket's full capacity instantly. Fixed-window counters are the simplest to implement but allow up to 2x the intended limit right at a window boundary (a burst just before the window ends, plus another right after it resets). Sliding-window-log is precise but its memory cost scales with request volume per client. Sliding-window-counter splits the difference — a weighted blend of the current and previous fixed windows — giving smooth behavior at roughly constant memory per key, which is why it's the common production default despite being a slight approximation.

**3. What happens when the shared store itself is unreachable.** This is the decision teams most often skip until an incident forces it. Failing open (letting all requests through when the limiter can't be reached) protects availability of the main service but removes the protection the limiter exists to provide, right when infrastructure is already unhealthy. Failing closed (rejecting all requests) protects downstream systems but can turn a limiter-store blip into a full outage for every client, including well-behaved ones. Most production systems fail open for short, transient store errors and treat sustained store unavailability as its own paged incident — explicitly choosing availability over strict enforcement for the common case, while still monitoring for the uncommon one.

## Tradeoff and failure mode worth calling out

The sharpest real-world failure mode is **hot-key contention on a single, extremely active client**. If one API key or IP is being hammered (a misbehaving integration, or an actual attack), every one of its requests maps to the same key in the shared store, and that key becomes a hot spot — a single partition or node in the distributed cache absorbing disproportionate load, even though the cluster as a whole has plenty of spare capacity. Consistent hashing distributes *different* keys well, but it does nothing to spread load for one pathologically popular key, since by definition that key always hashes to the same place. Production systems mitigate this with local, approximate pre-filtering — a cheap, short-lived in-memory check at the edge that can reject an obviously-over-limit client without even reaching the shared store — reserving the precise, centralized check for traffic that's plausibly within bounds. It's a reminder that "the rate limiter is distributed" doesn't automatically mean "load on the rate limiter is evenly distributed."
