---
title: "Consistency Patterns: Strong, Eventual, and the Space Between"
description: How CAP's consistency-vs-availability choice becomes strong, eventual, and causal consistency, plus read-your-writes and session guarantees.
roadmap: system-design
node: consistency-patterns
group: consistency-availability-and-consensus
video:
  provider: youtube
  id: "Y6Ev8GIlbxc"
  start: 0
  end: null
creator:
  name: Devoxx Poland
  channel_url: https://www.youtube.com/@DevoxxPoland
duration_seconds: 2940
level: intermediate
prerequisites:
  - acid-transactions-and-the-cap-theorem
takeaways:
  - CAP already forces a choice between consistency and availability during a partition — consistency patterns are the menu of options on the consistency side of that line.
  - Strong consistency makes every read see the latest committed write regardless of which replica answers, which usually means waiting on a leader or a quorum before responding.
  - Eventual consistency lets any replica answer immediately with whatever it has, trading a window of staleness for lower latency and higher availability.
  - Causal consistency keeps causally related operations, like a reply and the comment it replies to, in the right order everywhere, without paying for full strong consistency on unrelated writes.
  - Read-your-writes, monotonic reads, and session consistency are narrower, cheaper guarantees that fix specific user-facing glitches without upgrading an entire system to strong consistency.
  - Picking a consistency pattern is a per-feature decision, not a one-time database setting — a single application often mixes strong consistency for money and eventual consistency for a social feed.
check_questions:
  - q: Why might a user see their own just-published comment vanish after refreshing the page, even though the write actually succeeded?
    a: Because the refresh was answered by a different replica than the one the write landed on, and replication hadn't caught up yet — a violation of the read-your-writes guarantee.
  - q: What's the practical difference between eventual consistency and causal consistency?
    a: Eventual consistency makes no promises about the order in which unrelated updates become visible, while causal consistency specifically preserves the order of operations that depend on each other, such as a reply always appearing after the comment it replies to.
  - q: Why might an application use strong consistency for its payments table but eventual consistency for its "likes" counter?
    a: Because an incorrect balance is a real bug worth paying extra latency to prevent, while a like count that is briefly off by one is invisible to users and not worth slowing down every read to avoid.
related_formulas: []
---

## CAP already made you choose a side

The CAP theorem says a system facing a network partition has to choose between consistency and availability — consistency patterns are what that choice actually looks like once it turns into code and configuration. "Strong" and "eventual" consistency aren't abstract labels; they're specific, implementable guarantees with specific costs, and most real systems use several of them at once for different pieces of data.

## Strong consistency: one true answer, every time

A strongly consistent system guarantees that any read, from any node, returns the most recently committed write. There is no "stale" answer — only the current answer or an error. The cost is coordination: before a node can safely answer a read (or accept a write), it typically needs to confirm with a leader or a quorum of replicas that it actually has the latest value.

Consider a bank balance. If account A has $500 and two withdrawal requests for $400 each race in from two different ATMs, strong consistency is what prevents both from succeeding — whichever replica handles the second request has to know about the first withdrawal before it approves or denies the second one. That confirmation step costs latency, and if the network partitions, a strongly consistent node will refuse to answer rather than risk giving a stale balance. That refusal is CAP's "choose consistency over availability" in action.

## Eventual consistency: available now, correct eventually

An eventually consistent system drops the "confirm with everyone first" step. Any replica answers immediately with whatever value it currently holds, and if no new writes arrive, all replicas will eventually converge on the same value — but there's no bound on exactly when, and a read in the meantime might return something stale.

A product page's view counter is a reasonable candidate: if it briefly shows 1,204 views on one server and 1,207 on another, nobody is harmed, and the counter catches up seconds later. In exchange for that looseness, the system stays available even during a partition and answers reads faster, since no replica has to coordinate with anyone else before replying.

## Causal consistency: order where it matters, freedom where it doesn't

Causal consistency sits between the two. It doesn't demand that every node agree on a single global order of all writes — unrelated writes can still appear in different orders on different replicas — but it does guarantee that **causally related** writes appear in the right order everywhere. If a comment and a reply to that comment are causally linked, no replica is allowed to show the reply before the comment it replies to, even if other, unrelated comments on the same page can appear in a different order from one reader to the next.

This matters because eventual consistency alone doesn't protect against that specific failure: two unrelated writes racing to different replicas is harmless, but a reply outrunning its parent comment looks like a bug to the end user, even though strictly speaking "eventually consistent" was never violated. Causal consistency is cheaper to provide than strong consistency because it only has to track and preserve dependencies, not impose a single total order on everything.

## Client-centric guarantees: read-your-writes, monotonic reads, session consistency

Below the system-wide consistency model, there's a set of narrower guarantees aimed specifically at one client's experience, often implemented as a thin layer on top of an otherwise eventually consistent store:

- **Read-your-writes** guarantees that once a client writes a value, that same client's subsequent reads will see it — even if other clients might still see a stale value for a little longer. A common implementation: route a client's reads to the same replica (or the leader) for some window of time right after it writes.
- **Monotonic reads** guarantees that once a client has seen a value, it will never see an older value afterward — no going "backwards in time" because a later read happened to hit a more-lagging replica. This is usually enforced by pinning a client to a replica it's already read from, or by tracking a version number the client has already observed and refusing to serve anything older.
- **Session consistency** bundles both of the above (and sometimes more) into a single scope — "while this login session lasts, you get read-your-writes and monotonic reads" — which is often implemented via a sticky session or a client-carried version token, then relaxed back to plain eventual consistency once the session ends.

These exist because "eventually consistent, globally" is often fine for the system as a whole while still being a visibly broken experience for the one user who just made the change. Read-your-writes fixes exactly that gap without paying for strong consistency everywhere.

## Choosing per feature, not per database

The practical lesson is that "our database is eventually consistent" or "our database is strongly consistent" is the wrong level to make this decision at. A single e-commerce application reasonably uses strong consistency for inventory counts and payments (where being wrong costs money or trust), eventual consistency for recommendation feeds and view counters (where staleness is invisible), and a read-your-writes layer specifically for "did my profile edit actually save" moments, where a user watching their own change matters even if nobody else's view of it needs to update instantly. Consistency is a dial per feature, set as loose as the feature can tolerate and no looser.
