---
title: "Availability Patterns and Failover"
description: Active-passive vs active-active topologies, how failover is actually detected and triggered, and the split-brain risk when it goes wrong.
roadmap: system-design
node: availability-patterns-and-failover
group: consistency-availability-and-consensus
video:
  provider: youtube
  id: "CftcVhFMGSY"
  start: 0
  end: null
creator:
  name: CNCF [Cloud Native Computing Foundation]
  channel_url: https://www.youtube.com/@cncf
duration_seconds: 2054
level: intermediate
prerequisites:
  - consistency-patterns
  - replication-sharding-and-federation
takeaways:
  - Active-passive keeps one node serving traffic while one or more standbys wait idle, ready to take over; active-active runs multiple nodes serving traffic simultaneously.
  - Failover is only as good as its detection mechanism — a heartbeat or health check has to decide a node is dead before anything can route around it, and that decision takes real time.
  - A short, aggressive health-check timeout detects failure fast but risks declaring a merely slow node dead; a long, cautious timeout avoids false alarms but extends the outage before failover kicks in.
  - Split-brain happens when a failover decision is made wrong — two nodes each believe they are the one true primary at the same time, and both start accepting writes that will conflict.
  - Avoiding split-brain requires a majority-based decision (a quorum) rather than letting any single observer unilaterally declare a new primary.
  - Active-active avoids the idle-standby waste of active-passive but multiplies the consistency problems, since now every node that can accept a write has to agree with every other node about the current state.
check_questions:
  - q: What's the core tradeoff between active-passive and active-active topologies?
    a: Active-passive keeps a standby idle and ready, which wastes capacity but keeps conflict resolution simple since only one node ever accepts writes; active-active uses all nodes for traffic but has to resolve conflicts between simultaneous writes on different nodes.
  - q: Why can't a failover system just declare a node dead the instant it misses one heartbeat?
    a: Because a single missed heartbeat is often just a slow response or a dropped packet, not a real failure; declaring the node dead too eagerly triggers a failover for a node that was actually fine, which causes its own disruption.
  - q: How does split-brain actually happen, mechanically?
    a: A network partition or a slow health check makes some observers believe the primary is down, so they promote a standby to primary, while the original primary — still alive but unreachable to those observers — keeps believing it is still the primary too, leaving two nodes simultaneously accepting writes.
deeper:
  - label: "Day 1, Full Automatic Database: PostgreSQL HA with Kubernetes (KubeCon EU 2016)"
    url: "https://www.youtube.com/watch?v=iruaCgeG7qs"
related_formulas: []
---

## Two ways to survive a dead node

Every system that needs to stay up through a node failure has to answer the same question: what runs while the failed node is down? **Active-passive** keeps the answer simple — one node (the active, or primary) handles all real traffic, while one or more passive standbys replicate its state and sit idle, ready to be promoted. **Active-active** answers it differently: every node is already serving live traffic, so when one dies, the others simply absorb its share of the load without anyone needing to be "promoted" at all.

Active-passive is cheaper to reason about: there is only ever one place writes can land, so there's nothing to reconcile when the standby takes over — it just starts accepting writes where the old primary left off. The cost is waste: the standby's capacity sits unused until the day it's needed, and there is a gap, however short, between the primary dying and the standby being ready. Active-active eliminates both problems — no idle capacity, no promotion gap — but at the cost of needing every active node to agree on shared state, which reopens the consistency questions from the previous lesson. Multiple nodes accepting writes to the same data at the same time is exactly the scenario that forces a choice between strong consistency (coordinate every write across nodes, pay the latency) and eventual or conflict-resolved consistency (let nodes diverge briefly and reconcile after).

## Detecting failure before you can fail over

Neither topology helps if nothing knows the primary is down. That detection job falls to **heartbeats** and **health checks**: the primary (or an external monitor) periodically announces "I'm alive," and the absence of that signal for some threshold is what triggers a failover.

```
Primary ──heartbeat (every 2s)──▶ Monitor
Primary ──heartbeat (every 2s)──▶ Monitor
Primary ──heartbeat (every 2s)──▶ Monitor
                [ silence for 10s ]
Monitor: primary presumed dead → promote standby
```

The threshold is a real engineering tradeoff, not a formality. Set it too short — declare the primary dead after one missed heartbeat — and a single slow garbage-collection pause or a dropped network packet triggers an unnecessary failover, which has its own cost (a brief write unavailability, and now two nodes that both need to agree who's in charge). Set it too long, and a genuinely dead primary keeps the whole system down for longer than it needs to, because nothing has yet decided to act. Production systems typically require *several* consecutive missed checks, spaced a few seconds apart, before acting — trading a few extra seconds of downtime for confidence that the failure is real.

Health checks can also be smarter than "did it respond at all." A **liveness check** just confirms the process is running; a **readiness check** confirms it can actually serve traffic correctly — a database that's up but still replaying its write-ahead log on startup is alive but not ready, and a failover system that only checks liveness would route traffic to it anyway.

## When the failover decision itself is wrong: split-brain

The dangerous failure mode isn't "nothing notices the primary is down" — it's "different parts of the system disagree about whether it's down." A network partition is the classic trigger: the primary is still running and still believes it's in charge, but it's unreachable from the monitors (and possibly from the standby) on the other side of the partition. Those monitors, unable to reach it, correctly conclude they can't talk to the primary — and incorrectly conclude that means it's dead — so they promote the standby.

Now there are two nodes, each fully convinced it is the one true primary, both accepting writes. This is **split-brain**, and it's worse than an outage: an outage just means no writes happen; split-brain means two divergent, conflicting sets of writes happen, and someone has to reconcile them later, if that's even possible (two different customers might have been assigned the same just-allocated order number, for instance).

The standard defense is to never let a single observer unilaterally decide to promote a standby. Instead, promotion requires a **quorum** — a majority of the nodes in the cluster have to agree the primary is unreachable before a new one is promoted. A majority vote has a useful property: in a network split into two halves, at most one half can contain a majority, so at most one side can ever successfully promote a new primary. The minority side, even if it still has a primary that thinks it's in charge, loses the ability to accept writes once it can no longer reach a majority — which is precisely the kind of "refuse to answer rather than risk being wrong" choice CAP describes.

## Where this leads next

Quorum-based promotion is a specific, small instance of a much more general problem — getting a group of unreliable, possibly-partitioned nodes to agree on a single fact (who's primary, or more generally, what the current value of something is) without any single node being trusted to decide alone. That general problem is distributed consensus, and it's the subject of the next lesson.
