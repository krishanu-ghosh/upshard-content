---
title: "Case Study: Design a Key-Value Store"
description: A worked, Dynamo-style design for a distributed key-value store, covering partitioning, the replication/consistency tradeoff, and conflict resolution.
roadmap: system-design
node: design-a-key-value-store
group: case-studies
video:
  provider: youtube
  id: "6fOoXT1HYxk"
  start: 0
  end: null
creator:
  name: Aced (formerly Exponent)
  channel_url: https://www.youtube.com/@aced_io
duration_seconds: 2189
level: advanced
prerequisites:
  - distributed-consensus-paxos-and-raft
  - replication-sharding-and-federation
takeaways:
  - A distributed key-value store is, at its core, a hash table spread across many machines, so partitioning and replication aren't optional extras — they're the whole design problem.
  - Consistent hashing lets the cluster add or remove nodes while only remapping a small fraction of keys, instead of reshuffling everything the way naive modulo hashing would.
  - Every such system has to make an explicit CAP choice; most production key-value stores lean available and partition-tolerant, accepting eventual consistency rather than blocking writes during a network partition.
  - Tunable per-request consistency (quorum reads and writes, expressed as R and W thresholds against N replicas) lets the same store serve both "give me the fastest answer" and "give me the most consistent answer" callers.
  - Concurrent writes to the same key from different clients during a partition will produce conflicting versions that must be resolved somehow — either automatically (last-write-wins) or explicitly (vector clocks plus client-side or read-time reconciliation).
  - Replicating to N nodes doesn't just add durability — it also lets reads and writes be served by whichever replicas are healthiest and closest, which is part of how these stores stay available during partial failures.
check_questions:
  - q: Why does consistent hashing reduce the amount of data that must move when a node joins or leaves the cluster, compared to hashing keys modulo the number of nodes?
    a: Modulo hashing's node count appears directly in the formula, so changing it reassigns nearly every key to a different node, while consistent hashing maps both nodes and keys onto a fixed ring and only remaps the keys that fall in the range affected by the node being added or removed, leaving the rest of the ring's assignments untouched.
  - q: In a quorum system with N=3 replicas, why would a design choose W=2 and R=2 rather than W=1 and R=3 (or vice versa)?
    a: Any choice where R+W > N guarantees that a read quorum and a write quorum must overlap on at least one replica, so the read is guaranteed to see the latest acknowledged write; W=2,R=2 balances write and read latency evenly, while W=1,R=3 would make writes faster but reads slower (and more exposed to replica failures) and the reverse trade applies to W=3,R=1.
  - q: What problem do vector clocks solve that a simple "last write wins by timestamp" policy doesn't?
    a: Last-write-wins silently discards one of two concurrent writes based on wall-clock time, which can throw away real, independently-valid updates when clocks are skewed or writes truly happened concurrently, while a vector clock tracks causal history per replica, letting the system detect that two versions are genuinely concurrent (not one superseding the other) and hand the conflict to the application or a merge policy instead of guessing.
deeper:
  - label: "System Design Interview: Distributed Key-Value Store (Full HLD) — Hello HLD"
    url: "https://www.youtube.com/watch?v=0B4lkqMNvM8"
related_formulas: []
---

## Requirements

"Design a key-value store" usually means: design something in the spirit of Amazon's Dynamo, Apache Cassandra, or Redis Cluster — a `put(key, value)` / `get(key)` interface, but one that has to survive running across dozens or hundreds of unreliable machines.

Functional requirements:
- `put(key, value)`: store or update a value for a key.
- `get(key)`: retrieve the current value(s) for a key.
- Values are typically opaque blobs (small, often under a few hundred KB) — this isn't a query engine, it's a distributed hash table.

Non-functional requirements, which are really the whole design:
- **Scalability**: the dataset and request volume both need to scale horizontally by adding commodity nodes, not by buying bigger machines.
- **High availability**: the store should keep serving reads and writes even when some nodes are down or network-partitioned from each other.
- **Tunable consistency**: different applications calling the same store have different needs — some want the absolute latest value, some are fine with "probably recent."
- **Fault tolerance**: node failure is treated as the normal case, not the exception, given enough nodes running long enough.

## Capacity estimate

Suppose the store needs to hold 1 billion keys, each with a value averaging 1 KB, replicated 3 ways for durability.

- Raw data: 1B × 1 KB = 1 TB; replicated ×3 = 3 TB total across the cluster.
- If each node comfortably holds 500 GB of data (leaving headroom for compaction, indexes, and operational safety margin), that's roughly 6 nodes' worth of raw capacity — in practice teams run far more, smaller nodes, both for parallelism and so that losing any single node loses a small fraction of the keyspace.
- At 50,000 operations/second peak (a mix of gets and puts), and each node comfortably handling a few thousand ops/second for small-value workloads, the ops budget alone pushes toward a few dozen nodes — meaning throughput, not raw storage, is usually what determines cluster size in practice.

## High-level design

A client's request first needs to find which node(s) own a given key. Rather than a central lookup service (a single point of failure and a bottleneck), most designs hash the key onto a **consistent hashing ring** that also places every node, so any node — or a thin coordinator layer — can compute ownership locally with no coordination round-trip. The key's primary owner, plus the next N-1 nodes clockwise on the ring, hold its N replicas. A `put` request is coordinated by whichever node receives it: it forwards the write to all N replica nodes and waits for acknowledgment from at least W of them before telling the client the write succeeded. A `get` request similarly fans out to replicas and waits for at least R responses, then reconciles them (returning the latest, or all conflicting versions, depending on the conflict-resolution policy). Gossip protocols typically keep every node's view of cluster membership and ring layout eventually consistent, so the system keeps functioning correctly even as nodes join, leave, or fail.

## Key decisions

**1. The partitioning scheme.** Consistent hashing is close to the default answer here precisely because of the rebalancing cost it avoids: hashing keys modulo the node count means adding or removing a single node reshuffles almost the entire keyspace, which is operationally unacceptable at scale. On a consistent-hashing ring, a node joining or leaving only affects the keys in its immediate neighborhood on the ring — everything else stays put. Virtual nodes (each physical machine owning many smaller ranges scattered around the ring, rather than one contiguous arc) further smooth out load, so that one physical node leaving doesn't dump its entire range onto a single unlucky neighbor (see [replication, sharding, and federation](/roadmaps/system-design/replication-sharding-and-federation)).

**2. The replication/consistency tradeoff — this is the CAP decision made concrete.** With N replicas per key, the store exposes W (write quorum) and R (read quorum) as tunable parameters per request or per table. Choosing R + W > N guarantees that every read quorum overlaps with every prior write quorum on at least one replica, which guarantees the read sees the latest acknowledged write — strong consistency, at the cost of blocking until enough replicas respond, which is painful during a partition. Choosing R + W ≤ N (e.g., W=1, R=1) sacrifices that guarantee for much lower latency and availability even when some replicas are unreachable — a client might read stale data, but it never blocks waiting for replicas that can't be reached. Dynamo-style stores default to leaning available: writes succeed quickly against a minority quorum, and the system resolves any resulting disagreement later (see [consistency patterns](/roadmaps/system-design/consistency-patterns) and [distributed consensus](/roadmaps/system-design/distributed-consensus-paxos-and-raft) for the mechanisms that make a *stronger* choice possible when a use case genuinely needs it).

**3. Conflict resolution on concurrent writes.** Once R + W ≤ N is on the table, concurrent writes to the same key during a partition will produce genuinely conflicting versions — not a bug, an inherent consequence of accepting writes without full agreement. Last-write-wins (pick whichever write has the latest timestamp) is simple and automatic but can silently drop a real update if clocks are skewed or two writes were truly concurrent. Vector clocks track causal "happened-before" relationships per replica instead of wall-clock time, so the system can distinguish "this write truly supersedes that one" from "these two writes are concurrent and neither wins automatically" — in the second case, it returns both versions to the client (or an application-level merge function) to reconcile, which is more correct but pushes real complexity onto whoever calls `get`.

## Tradeoff and failure mode worth calling out

The tradeoff worth internalizing is that **weakening consistency to gain availability doesn't remove the conflict problem — it just relocates it** from the storage layer to the application layer. A store running with R=1, W=1 will absolutely return stale reads and will absolutely need to reconcile concurrent writes eventually; choosing that configuration is really choosing to make "the client (or a background anti-entropy process) handles reconciliation" someone's explicit job, rather than assuming the storage engine will quietly make conflicts disappear. Teams that adopt a Dynamo-style store for its availability properties and then write application code that silently assumes `get` always returns one unambiguous value are setting up a correctness bug that only surfaces during a real partition — exactly the moment the system is already under the most stress.
