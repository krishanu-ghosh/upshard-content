---
title: "Distributed Consensus: Paxos and Raft"
description: Why distributed systems need to agree on one value despite failures, and the core ideas behind Paxos and Raft at a conceptual level.
roadmap: system-design
node: distributed-consensus-paxos-and-raft
group: consistency-availability-and-consensus
video:
  provider: youtube
  id: "6bBggO6KN_k"
  start: 0
  end: null
creator:
  name: CoreOS
  channel_url: https://www.youtube.com/@CoreOS
duration_seconds: 2409
level: advanced
prerequisites:
  - acid-transactions-and-the-cap-theorem
takeaways:
  - Consensus means getting a group of nodes to agree on a single value or a single sequence of operations, even though some nodes might crash or the network might drop messages.
  - A naive "ask everyone and take the first answer" approach breaks the moment two nodes propose conflicting values at nearly the same time, which is exactly the situation consensus algorithms are built to resolve safely.
  - Both Paxos and Raft rely on majority quorums — a value or a leader is only considered accepted once more than half the nodes agree, which guarantees any two quorums overlap in at least one node.
  - Raft splits the problem into two separate, easier-to-reason-about pieces — leader election and log replication — which is precisely why it's considered easier to understand and implement than classic Paxos.
  - Once a cluster has a stable, uniquely-elected leader, agreement becomes simple — the leader just replicates its log to a majority of followers in order, and a quorum's acknowledgment makes an entry durable and committed.
  - Consensus is how systems like etcd, ZooKeeper, and Kubernetes' control plane stay correct during network partitions and node failures, by making leader election and configuration changes agree on one answer instead of silently diverging.
check_questions:
  - q: Why can't a distributed system just let any node unilaterally decide "I'm the leader now" when it loses contact with the current leader?
    a: Because a network partition could make several nodes simultaneously lose contact with the leader and with each other, so if each one declared itself leader independently, the cluster would end up with multiple leaders accepting conflicting writes at once.
  - q: Why does requiring a majority (quorum) specifically — not just "more than one node agrees" — prevent two conflicting values from both being accepted?
    a: Because any two majorities out of the same set of nodes must overlap in at least one node, and that overlapping node can only have acknowledged one of the two conflicting values, so the second one is blocked from also reaching a majority.
  - q: What two sub-problems does Raft split consensus into, and why does that split make it easier to reason about than classic Paxos?
    a: Leader election (agreeing on who is in charge) and log replication (the leader copying entries to followers in order); separating them means each piece can be understood and verified largely on its own, instead of reasoning about one tangled algorithm that does both at once.
deeper:
  - label: "Dr. TLA+ Series: Raft"
    url: "https://www.youtube.com/watch?v=6Kwx8zfGW0Y"
related_formulas: []
---

## Agreeing on one answer when nodes can fail

A single machine never has to "agree" with itself about anything — whatever it last wrote is the truth. A cluster of machines does, and that's a much harder problem the moment any machine can crash, restart, or simply be slow to respond. **Consensus** is the name for getting a group of nodes to settle on one agreed value (or one agreed order of operations) despite some of them failing or the network between them misbehaving — and it turns out to be one of the foundational, genuinely hard problems in distributed systems, which is exactly why two specific algorithms, Paxos and Raft, dominate how real systems solve it.

The naive approach — "send your proposed value to everyone, and whoever gets a response first wins" — collapses immediately under concurrency. If two nodes each believe they should be the leader and both start proposing values at nearly the same moment, different nodes can end up accepting different proposals, and now the cluster disagrees about something it absolutely cannot afford to disagree about (who the leader is, or what the next entry in a replicated log should be).

## The quorum trick that makes agreement safe

Both Paxos and Raft solve the conflicting-proposals problem with the same core trick: nothing is considered decided until a **majority** of the cluster's nodes — a quorum, more than half — has accepted it. This one requirement has a property worth sitting with: in a cluster of 5 nodes, any majority has at least 3 members, and any two majorities out of the same 5 nodes are mathematically guaranteed to overlap in at least one node. That overlap is what prevents two different values from both reaching majority acceptance — the overlapping node can only have voted for one of them, so the second proposal can get at most a minority and is blocked.

```
5 nodes, quorum = 3

Majority A: {1, 2, 3}
Majority B: {3, 4, 5}
                ↑
        node 3 is in both —
        it can't have accepted two different values
```

This is also why consensus clusters are deployed in odd numbers (3, 5, 7) rather than even ones — an even-sized cluster can split exactly in half during a partition, with neither half holding a majority, which just stalls progress rather than risking two conflicting majorities (a safer failure, but still worth avoiding by sizing clusters oddly).

## Paxos: correct first, understandable second

Paxos, formalized by Leslie Lamport, was the first widely accepted general solution to this problem, and it works by having nodes play two overlapping roles — **proposers**, who suggest values, and **acceptors**, who vote on them — through a two-phase protocol (roughly: "has anyone proposed something already, and if so what" followed by "here's my value, accept it"). Paxos is correct and provably safe, but it earned a reputation for being difficult to fully understand and even harder to implement correctly from the paper alone, largely because basic ("single-decree") Paxos only agrees on one value, and real systems need a continuously replicated log of many values — turning that single-value agreement into "multi-Paxos" introduces a lot of subtlety that the original paper leaves as an exercise.

## Raft: the same guarantees, split into two problems

Raft, designed explicitly to be "Paxos, but understandable," reaches equivalent guarantees by decomposing the problem into two pieces that can mostly be reasoned about separately:

- **Leader election**: at any given time, the cluster has at most one leader. Followers expect periodic heartbeats from the leader; if a follower doesn't hear one within a randomized timeout, it becomes a candidate, increments a term number, and requests votes from the rest of the cluster. A candidate that wins a majority vote becomes the new leader for that term. The randomized timeout is a small but important detail — if every follower's timeout were identical, they'd all become candidates simultaneously and split the vote repeatedly.
- **Log replication**: once a leader exists, every client write becomes an entry appended to the leader's log, which the leader then pushes out to its followers. An entry is only considered **committed** — safe, durable, and visible — once a majority of nodes have stored it. The leader keeps followers' logs consistent by walking backward to find the last point of agreement whenever a follower's log has diverged, then overwriting anything after that point with its own.

Because leadership is clearly defined and exclusive, log replication during normal operation is almost boring: the leader just tells everyone what to append, in order. Nearly all of Raft's real complexity lives in handling the edges — a leader crashing mid-replication, a network partition isolating a minority, or a follower rejoining after being gone for a while — and even there, Raft's answer is consistent: nothing is final until a majority has it, and a new leader's election term always wins out over any leftover state from an older term.

## Why this matters beyond the algorithm itself

Consensus isn't an academic exercise bolted onto real infrastructure — it's the mechanism underneath tools many systems already depend on. etcd (Kubernetes' own configuration store) and Apache ZooKeeper both run Raft or Paxos-family algorithms internally specifically so that "who is the current leader" and "what is the current configuration" never silently diverge across a cluster, even while individual nodes come and go. The quorum-based promotion from the previous lesson's split-brain discussion isn't a separate idea — it's consensus, applied to exactly one question: who gets to be primary.
