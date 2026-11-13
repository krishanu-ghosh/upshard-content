---
title: "Leader Election"
description: Practical leader-election mechanisms — the bully algorithm and lease-based election via etcd/ZooKeeper-style ephemeral nodes and TTLs.
roadmap: system-design
node: leader-election
group: reliability-and-resilience-patterns
video:
  provider: youtube
  id: "3SnPwhKvNZ0"
  start: 0
  end: null
creator:
  name: sudoCODE
  channel_url: https://www.youtube.com/@sudocode
duration_seconds: 527
level: intermediate
prerequisites:
  - distributed-consensus-paxos-and-raft
takeaways:
  - Leader election picks exactly one node out of a group to coordinate a task — like assigning work or owning a write path — so the others don't collide doing it simultaneously.
  - The bully algorithm elects the highest-ID live node as leader by having any node that notices the leader is gone message all higher-ID nodes and yield to the first one that answers.
  - Lease-based election avoids an explicit election protocol entirely — nodes race to create an ephemeral key with a short TTL, and whoever creates it first is leader until the lease expires or is renewed.
  - An ephemeral node or lease is tied to its holder's active session, so if the leader crashes or loses its connection, the coordination service removes the lease automatically — no separate failure-detection protocol needed.
  - Lease-based election is what etcd and ZooKeeper are actually used for in practice, because they already provide the consistent, replicated storage that makes "exactly one node can hold this key" a safe guarantee.
  - A leader must keep renewing its lease before it expires, and must stop acting as leader the instant it fails to renew in time, because a network partition can let a new leader be elected while the old one still thinks it holds the lease.
check_questions:
  - q: In the bully algorithm, why does the election always converge on the highest-ID surviving node specifically, rather than any arbitrary node?
    a: Because any node that starts an election contacts all nodes with a higher ID and defers to the first one that responds, so the only node that is never deferred away from is the single highest-ID node that's still alive — every other live node eventually hears from, and yields to, something higher.
  - q: Why is a lease-based leader election using etcd or ZooKeeper generally preferred over implementing the bully algorithm from scratch?
    a: Because etcd and ZooKeeper already solve the hard part — a consistent, replicated store where "only one client can hold this key" is a safe, strongly-consistent guarantee backed by a real consensus protocol — so leader election becomes a simple create-if-absent operation with a TTL, instead of requiring you to implement and get right your own failure detection and message-passing protocol.
  - q: A leader holds a lease with a 10-second TTL and renews it every 3 seconds. It experiences a 12-second network partition that cuts it off from the coordination service. What has to happen, and why does it matter?
    a: The lease expires after 10 seconds without a renewal, so the coordination service lets another node acquire it and become the new leader — critically, the old leader must recognize it failed to renew in time and stop acting as leader itself, otherwise there would briefly be two nodes both believing they're the leader at once.
---

## Why you need exactly one

Plenty of distributed coordination problems only make sense if exactly one node is doing them: assigning partitions to workers, owning the write path for a piece of data, running a periodic cleanup job that would cause duplicate work (or duplicate side effects) if two instances ran it at once. Leader election is the mechanism that picks that one node out of a group and keeps the choice correct as nodes come and go. The `distributed-consensus-paxos-and-raft` lesson in this roadmap covers *why* agreement is even possible in the presence of failures and network delay — majority quorums, log replication, the guarantees Raft and Paxos provide. This lesson is about the practical side: given that a consensus-backed system exists, what does actually running a leader election look like in code you might write or operate?

## The bully algorithm

The bully algorithm is one of the oldest and most intuitive leader election protocols, and it's a good mental model even though production systems rarely implement it directly anymore. Every node has a unique, comparable ID (often just a node number or address). The rule is simple: the leader is always the highest-ID node that's currently alive.

When a node notices the current leader is unresponsive (its heartbeat stopped, say), it starts an election by sending an election message to every node with a *higher* ID than itself. Two things can happen:

- If no higher-ID node responds within a timeout, this node declares itself the leader and announces that to everyone.
- If a higher-ID node does respond, that node takes over starting its own election (sending to nodes higher than *it*), and the original node simply waits to be told who won.

This cascades until the single highest-ID live node either hears from nothing higher (and wins) or, if it was the one that failed, the next-highest node wins instead. The name comes from this behavior: a higher-ID node always "bullies" a lower-ID one out of the running the moment it responds. It's simple to reason about and guarantees a deterministic winner, but it has real costs — it can require O(n²) messages in the worst case during an election storm, and if the highest-ID node is flapping (going up and down repeatedly), you get repeated, disruptive re-elections.

## Lease-based election with ephemeral keys and TTLs

The approach actually used in most real systems today sidesteps an explicit voting protocol entirely by piggybacking on a strongly consistent coordination service — etcd, ZooKeeper, or Consul — that already guarantees "only one client can successfully create this specific key at a time," because the service itself is built on a consensus protocol like Raft under the hood.

The mechanics: every candidate node attempts to create the same key (e.g., `/election/leader`) as part of a *lease* with a short time-to-live, say 10 seconds. The coordination service only lets one of them succeed — that's the strong consistency guarantee doing the work — and whichever node succeeds is the leader. Every other candidate's attempt fails, and instead of giving up, each failed candidate sets a watch on that key so it's notified the moment it disappears.

The leader must keep renewing its lease — refreshing the TTL — before it expires, typically well before the deadline (e.g., every 3 seconds against a 10-second TTL) to leave margin for network jitter. If the leader crashes, hangs, or simply loses connectivity to the coordination service, it stops renewing, the lease's TTL runs out, and the coordination service deletes the key on its own — no heartbeat protocol, no explicit failure detector needed, because the lease mechanism *is* the failure detector. Every watcher gets notified the key is gone and races to create it again; one of them wins and becomes the new leader.

This is substantially simpler to build correctly than the bully algorithm, because the hard distributed-systems problem — making sure everyone agrees on who holds the key, even while nodes fail and messages get delayed — has already been solved inside etcd/ZooKeeper's own consensus layer. Your application code just does a create-if-absent-with-TTL operation and a watch; it doesn't implement an election protocol at all.

## The danger that doesn't go away: split brain

Lease expiry solves failure detection, but it introduces its own sharp edge. If the old leader is merely partitioned from the coordination service rather than actually dead, it may still be alive and believe it's the leader right up until it notices its renewal calls are failing — meanwhile the lease has already expired on the coordination service's side, a new leader has been elected, and for some window there are genuinely two nodes that each believe they hold the leadership. The discipline that prevents this from causing damage is: a leader must treat a failed lease renewal as an immediate instruction to stop acting as leader, before it does anything else, rather than continuing to act on stale authority while it figures out what went wrong. Any action a leader takes that has external, hard-to-undo consequences should additionally be guarded by checking its lease is still valid (often via a monotonically increasing "fencing token" tied to the lease) at the moment it acts, not just at the moment it was elected.
