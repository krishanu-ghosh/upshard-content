---
title: "Distributed Caching Systems"
description: How Redis/Memcached-style caches spread keys with consistent hashing, and how locking, coalescing, and jitter stop a cache stampede.
roadmap: system-design
node: distributed-caching-systems
group: caching-at-scale
video:
  provider: youtube
  id: "UF9Iqmg94tk"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 484
level: intermediate
prerequisites:
  - caching-strategies-at-scale
takeaways:
  - A single Redis or Memcached instance runs out of memory and throughput long before a large system's cache needs do, so production caches are usually clusters of many nodes, each holding a shard of the total keyspace.
  - Naive key placement with hash(key) % N falls apart the moment N changes — adding or removing a single node remaps almost every key, instantly invalidating nearly the whole cache cluster.
  - Consistent hashing places both nodes and keys on a hash ring, so a key always maps to the next node clockwise from it; adding or removing one node only remaps the keys that land between it and its neighbor, leaving everything else untouched.
  - Virtual nodes fix consistent hashing's tendency toward uneven load when there are few physical nodes, by giving each physical node many positions on the ring instead of just one, spreading its share of the keyspace more evenly.
  - A cache stampede happens when a single hot key expires and many concurrent requests all miss at the same instant, hammering the backing store with a sudden multiple of its normal load; locking (only one request rebuilds the value), request coalescing (concurrent misses share one in-flight rebuild), and jittered TTLs (randomizing expiry so keys don't all die together) are the standard defenses.
check_questions:
  - q: Why does hash(key) % N break down as a way to place keys across a cache cluster?
    a: Because the modulus N is the number of nodes, changing N by adding or removing even one node changes the result of the modulo operation for almost every key, which means almost every key now maps to a different node than before and the cache is effectively invalidated cluster-wide.
  - q: How does consistent hashing limit the damage when a node is added or removed?
    a: Nodes and keys both get positions on a hash ring, and each key belongs to the next node going clockwise from it, so removing or adding a node only affects the keys that fall in the arc between that node and its nearest neighbor — every other key's nearest clockwise node is unchanged, so only a small, predictable fraction of keys need to move.
  - q: What is a cache stampede, and what's one standard way to prevent it?
    a: It's what happens when a single popular key expires and many concurrent requests miss on it at the same moment, all trying to rebuild it from the backing store simultaneously and multiplying the load that store sees; a standard fix is request coalescing or locking — letting only one of those concurrent requests actually rebuild the value while the others wait for that one rebuild to finish and reuse its result.
deeper:
  - label: "Cache Stampede: How One Expired Key Crashes Your DB"
    url: "https://www.youtube.com/watch?v=TAZGA-aScPA"
related_formulas: []
---

## One cache node isn't enough

A single Redis or Memcached process can serve a lot of traffic, but it has finite memory and finite throughput, and a large system's working set of frequently-accessed data — user sessions, hot query results, rendered fragments — routinely exceeds what one machine can hold or serve fast enough. The standard answer is a **distributed cache**: a cluster of cache nodes, each holding a shard of the overall keyspace, with client libraries or a routing layer deciding which node owns a given key. Redis Cluster and Memcached-with-client-side-sharding are the two most common concrete shapes this takes, but the underlying placement problem is the same either way, and it's harder than it first looks.

## Why naive hashing falls apart

The obvious way to decide which of N nodes owns a key is `hash(key) % N`. It's simple, and it distributes keys evenly across N nodes as long as N doesn't change. The problem is that N *does* change — nodes get added for capacity, or removed after a failure — and the moment N changes from, say, 4 to 5, the result of `hash(key) % N` changes for nearly every key, not just the ones that logically should move. A key that used to map to node 2 might now map to node 0, even though node 2 never went anywhere. From the cache's point of view, this looks like almost the entire cluster's contents simultaneously becoming "wrong" — every client now looks for its data on a different node than where it was stored, so nearly every lookup misses, and the backing database gets hit with something close to the cache's entire normal traffic all at once.

## Consistent hashing: only move what has to move

**Consistent hashing** solves this by decoupling "which node owns this key" from the total node count. Both nodes and keys are hashed onto the same circular space — a **hash ring**, typically spanning a fixed range like 0 to 2³²−1. Each node is placed at the ring position its hash produces; each key is placed the same way, and a key belongs to whichever node's position is the first one reached going clockwise from the key's position.

The payoff shows up exactly when the node count changes. If a new node is inserted at some position on the ring, it only takes over the keys in the arc between itself and the next node clockwise from its old neighbor — every other key on the ring still maps to the same node it always did, because nothing about *their* nearest-clockwise-node changed. Removing a node works the same way in reverse: only the keys that belonged to it get reassigned, to whichever node is now the next one clockwise, and the rest of the ring is untouched. Instead of "changing N remaps almost everything," it becomes "changing N remaps only the keys near the change" — roughly `1/N` of the keyspace moves on average, not nearly all of it.

## Virtual nodes: fixing uneven load on the ring

Plain consistent hashing has its own wrinkle: with only a handful of physical nodes, their positions on the ring can land unevenly by chance — two nodes placed close together leave a big gap to the next one, so whichever node owns that gap ends up responsible for a disproportionate share of the keyspace. **Virtual nodes** fix this by giving each physical node many positions on the ring instead of one — a cluster of 4 physical nodes might place 100 virtual points per node, 400 points total, scattered around the ring. Keys still map to the nearest virtual point clockwise, but because each physical node now owns many small, scattered arcs instead of one big one, the law of averages takes over and each physical node ends up with a roughly equal total share of the keyspace, even though the individual placements were random. This is the detail that makes consistent hashing practical in real cache clusters rather than just a clean idea that works best with specific node counts.

## Cache stampede: when a hot key expiring becomes an incident

Sharding and placement solve "which node holds this key," but a distributed cache has a separate, very practical failure mode: the **cache stampede** (also called thundering herd or the dogpile effect). Picture a single key — a trending article's rendered page, a frequently-read configuration value — being requested hundreds of times per second. Its TTL expires. In the instant after expiry, every one of those hundreds of concurrent requests misses the cache at the same time, and if each one independently goes and rebuilds the value from the database, the database just got hit with hundreds of simultaneous queries for the exact same data it would normally serve once. For a sufficiently hot key, this spike alone can be enough to take down a database that was otherwise comfortably within capacity.

Three mitigations handle this in practice, and they're often combined:

- **Locking / request coalescing:** when a request misses, it checks whether a rebuild for that key is already in progress; if so, it waits for that in-flight rebuild to finish and reuses its result instead of starting a duplicate one. Only one request actually touches the database per expired key, no matter how many concurrent requests needed it.
- **Jittered TTLs:** instead of setting every related key to expire at exactly the same instant (e.g., a batch of keys all set with `ttl=300` at the same moment), add a small random offset to each one (`ttl=300 + random(-30, 30)`). Keys then expire at slightly staggered times instead of all at once, so a stampede on one key doesn't coincide with stampedes on many others, and even a single key's own expiry is less likely to line up exactly with a traffic spike.
- **Refresh-ahead**, covered in the previous lesson, sidesteps the problem differently: if a hot key is refreshed proactively before it ever expires, there's no moment where it's simultaneously gone and in-demand, so there's nothing for a stampede to form around.

None of these are exotic — they're standard operating practice for any distributed cache serving a workload with genuinely hot keys, and skipping them is one of the more common ways an otherwise well-designed caching layer ends up causing the exact database overload it was built to prevent.
