---
title: "Case Study: Design a Distributed Cache"
description: A worked design for a Redis/Memcached-style distributed cache, covering eviction, cross-node invalidation, and resizing without a full reshuffle.
roadmap: system-design
node: design-a-distributed-cache
group: case-studies
video:
  provider: youtube
  id: "DbT5mNm47Sg"
  start: 0
  end: null
creator:
  name: Interview Kickstart US
  channel_url: https://www.youtube.com/@Interviewkickstart
duration_seconds: 2060
level: advanced
prerequisites:
  - distributed-caching-systems
  - consistency-patterns
takeaways:
  - A distributed cache trades the durability of a database for the speed of memory, so its whole design should be read as a set of answers to "what happens when this cached copy is wrong or gone?".
  - Consistent hashing is what lets the cache cluster grow or shrink without remapping (and therefore cold-starting) most of its keys.
  - Eviction under memory pressure and invalidation on data change are different problems with different triggers, even though both end with a key leaving the cache.
  - A write-through or write-invalidate policy keeps the cache closer to correct at the cost of extra write latency; a lazily-expiring, TTL-only policy is faster to write but tolerates staler reads.
  - Hot keys (one item getting a disproportionate share of traffic) break the load-balancing promise of consistent hashing, because that one key always lands on the same node no matter how evenly the hash ring is built.
  - Because a cache is explicitly allowed to lose data, every design has to define what happens on a cache miss or a cold cache — the system must still be correct, just slower, when the cache is empty.
check_questions:
  - q: Why does consistent hashing matter more for a cache than for a typical sharded database?
    a: A cache is explicitly disposable — losing cached data costs latency, not correctness — but that disposability is only a feature if resizing the cluster doesn't itself destroy most of the useful cached data at once, and consistent hashing is what limits a resize to remapping a small fraction of keys instead of invalidating nearly everything, the way modulo hashing would.
  - q: What's the difference between the trigger for cache eviction and the trigger for cache invalidation?
    a: Eviction is triggered by the cache running low on memory and needing to free space, typically guided by a policy like LRU that picks the least-useful item to remove regardless of whether the underlying data changed, while invalidation is triggered by the underlying data actually changing, removing or updating a specific cached entry so clients stop reading a now-stale value even if there's plenty of free memory.
  - q: Why can't consistent hashing alone prevent a hot-key problem?
    a: Consistent hashing distributes different keys across nodes roughly evenly, but a single specific key always maps to the same node by definition, so if that one key receives a disproportionate share of all traffic, the node owning it becomes a hot spot no matter how well-distributed the rest of the keyspace is.
deeper:
  - label: "Caching in System Design Interviews w/ Meta Staff Engineer (Hello Interview)"
    url: "https://www.youtube.com/watch?v=1NngTUYPdpI"
related_formulas: []
---

## Requirements

A distributed cache (think Redis Cluster or Memcached with consistent-hashing clients) sits in front of a slower system of record and serves hot data from memory across many nodes. Unlike the previous case study's key-value store, this system is explicitly allowed to lose data — that's the whole point of calling it a cache rather than a store.

Functional requirements:
- `get(key)` / `set(key, value, ttl)` with sub-millisecond latency.
- Automatic eviction when memory fills up.
- Invalidation (or update) of specific entries when the underlying data changes.

Non-functional requirements:
- **Very low, predictable latency**: this is the entire reason the cache exists; a cache that's merely "fast-ish" doesn't justify its own operational complexity.
- **Horizontal scalability**: both capacity (more RAM) and throughput (more ops/second) need to grow by adding nodes.
- **Graceful degradation, not strict correctness**: a stale or missing cache entry should make the system slower, never wrong in a way that matters — which pushes genuinely strong consistency requirements back onto the system of record, not onto the cache.
- **Resizable without a stampede**: adding or removing nodes shouldn't evict so much of the cache at once that the backing store gets hit with a thundering herd of requests it wasn't sized for.

## Capacity estimate

Suppose the service wants to cache the hottest 10% of a 500-million-row dataset, with each cached value averaging 2 KB, and needs to sustain 200,000 cache operations/second at peak.

- Working set: 500M × 10% × 2 KB = 100 GB of data that should live in cache at any given time.
- If each cache node has 32 GB of usable memory (leaving headroom below the box's total RAM for overhead), that's roughly 4 nodes just for capacity — but throughput usually dominates the real sizing decision.
- At 200,000 ops/second and, say, 50,000 ops/second per node for simple get/set workloads, throughput alone pushes toward at least 4-5 nodes — meaning capacity and throughput estimates land in a similar range here, which isn't always true and is worth checking explicitly rather than assuming.
- Replication for availability (losing one node shouldn't blank out a fifth of the cache and immediately slam the backing store) adds further nodes on top of this baseline.

## High-level design

Application servers hold a cache client that knows the consistent-hashing ring layout (either computed locally from cluster membership, or fetched from a lightweight coordination service). On a `get`, the client hashes the key to find its owning node directly and queries it — no proxy hop needed, which keeps latency low. On a miss, the application server falls through to the backing database, computes or fetches the value, writes it back into the cache, and returns it to the caller — the standard cache-aside pattern. On a `set` driven by an application write (not just a cache-fill), the system either writes through to the cache synchronously, or — more commonly — simply invalidates (deletes) the now-stale key and lets the next reader repopulate it lazily. A background eviction process on each node independently enforces a memory ceiling using an eviction policy like LRU, freeing space for new entries without needing cluster-wide coordination.

## Key decisions

**1. Eviction under memory pressure.** Every cache node eventually fills up, and picking what to evict is a purely local decision each node makes about its own memory, independent of what any other node is doing. LRU (evict whatever hasn't been read recently) suits general-purpose workloads well; LFU suits workloads where a long history of heavy use should protect an item even through a brief lull. The real design question isn't just "which policy" — it's making sure eviction is cheap (O(1), via a hash map plus a doubly-linked list for LRU) so that the act of freeing memory doesn't itself become a latency spike on the hot path.

**2. Cache invalidation across nodes.** This is widely considered one of the two genuinely hard problems in computing for good reason: once a value is cached on potentially many nodes (or many layers — a local in-process cache, a shared cluster, a CDN), updating the source of truth has to somehow tell every copy it's stale. Write-invalidate (delete the cached key on write, let the next read repopulate it) is simpler and avoids ever serving wrong data for long, at the cost of a guaranteed cache miss on the next read. Write-through (update the cache and the database together, synchronously) avoids that miss but adds write latency and risks the two falling out of sync if the write to one side fails after the other succeeds. TTL-based expiration is a blunt backstop used alongside either approach — even if explicit invalidation is missed or racy, no entry lives wrong forever (see [consistency patterns](/roadmaps/system-design/consistency-patterns) for the broader tradeoff this sits inside).

**3. Consistent hashing for resizing without a full reshuffle.** Adding a node to grow capacity, or removing one after a failure, has to move as little data as possible — not because the data is precious (it's a cache), but because moving a lot of data at once means a lot of cache misses at once, and those misses all funnel straight to the backing store simultaneously. Consistent hashing with virtual nodes keeps a resize's blast radius proportional to the one node changing, not the whole cluster, which is what keeps the backing database from getting slammed every time the cache cluster is scaled.

## Tradeoff and failure mode worth calling out

The failure mode worth internalizing is the **cold-cache stampede**: when a popular key expires, or a whole node is lost and its keys become unavailable, many concurrent requests can all miss the cache for the same key at the same moment and all fall through to the backing store simultaneously — turning one expensive computation into dozens or hundreds of identical, simultaneous expensive computations, right as the system is already degraded. The common mitigation is request coalescing (the first miss computes the value while concurrent requests for the same key wait on that one in-flight computation instead of starting their own) or staggered TTLs with jitter (so a batch of entries cached at the same time doesn't all expire in the same instant). It's a direct consequence of the cache's core bet — tolerate losing data for speed — colliding with real-world traffic concentration, and it's exactly the kind of thing that looks fine in a design doc and only shows up under the load a cache was built to absorb in the first place.
