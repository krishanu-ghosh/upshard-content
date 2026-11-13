---
title: "Distributed Transactions: Two-Phase Commit and Sagas"
description: Why two-phase commit blocks when its coordinator fails, and how the saga pattern trades atomicity for availability with compensating transactions.
roadmap: system-design
node: distributed-transactions-2pc-and-sagas
group: consistency-availability-and-consensus
video:
  provider: youtube
  id: "7DoT2sTGulc"
  start: 0
  end: null
creator:
  name: Jordan Has No Life
  channel_url: https://www.youtube.com/@jordanhasnolife5163
duration_seconds: 499
level: advanced
prerequisites:
  - acid-transactions-and-the-cap-theorem
  - distributed-consensus-paxos-and-raft
takeaways:
  - A distributed transaction has to get multiple separate services or databases to either all commit the same logical change or all roll it back — plain local ACID transactions can't do this across service boundaries.
  - Two-phase commit (2PC) has a coordinator ask every participant to "prepare" and lock its resources first, and only tells everyone to actually commit once every participant has said yes.
  - 2PC's fatal weakness is blocking — if the coordinator crashes after participants have prepared but before it sends the commit decision, every participant is stuck holding its locks indefinitely, unable to safely commit or abort on its own.
  - The saga pattern replaces one big distributed transaction with a sequence of local transactions, each with its own predefined compensating transaction that undoes it if a later step fails.
  - Sagas can be coordinated two ways — orchestration, where a central coordinator tells each service what to do next, or choreography, where each service reacts to events and decides its own next step with no central coordinator.
  - Sagas trade strict atomicity for availability and throughput — the data can be briefly, visibly inconsistent mid-saga, which is the deliberate cost of not blocking services the way 2PC does.
check_questions:
  - q: In two-phase commit, what exactly happens during the "prepare" phase, before anything is actually committed?
    a: The coordinator asks every participant to do everything needed to guarantee it can commit — writing the change durably and locking the relevant resources — and each participant replies yes or no without yet making the change visible or final.
  - q: Why does a coordinator crash between the prepare and commit phases cause participants to block rather than just abort safely?
    a: Because a participant that voted yes has promised it can commit and is not allowed to unilaterally abort — it doesn't know whether the coordinator already told other participants to commit, so guessing wrong in either direction could leave the system inconsistent, and its only safe option is to keep its locks and wait.
  - q: How does a saga avoid the blocking problem that 2PC has?
    a: Each step in a saga is its own independent local transaction that commits immediately rather than waiting on a cross-service lock, so no service ever sits blocked holding resources on behalf of a transaction some other, possibly-crashed service hasn't finished — a failure is instead handled afterward by running compensating transactions for whatever already committed.
related_formulas: []
---

## When one transaction isn't on one database

A single ACID transaction works because one database can make atomicity a local promise — it controls its own log and its own locks, so "all or nothing" is entirely within its power to guarantee. The moment a logical operation spans multiple services, each with its own database (place an order: reserve inventory in one service, charge a card in another, schedule a shipment in a third), no single database can make that same all-or-nothing promise on its own. Something has to coordinate across all of them, and there are two fundamentally different answers to how: two-phase commit, and the saga pattern.

## Two-phase commit: agree first, commit second

**Two-phase commit (2PC)** introduces a **coordinator** that drives every participant through two rounds:

```
Phase 1 — Prepare
Coordinator → Participant A: "Can you commit?"  → A locks resources, replies YES
Coordinator → Participant B: "Can you commit?"  → B locks resources, replies YES
Coordinator → Participant C: "Can you commit?"  → C locks resources, replies YES

Phase 2 — Commit (only if ALL replied YES)
Coordinator → A: "Commit"
Coordinator → B: "Commit"
Coordinator → C: "Commit"
```

In the prepare phase, every participant does everything short of actually finalizing the change: it writes the pending change durably and takes out locks on whatever it's about to modify, then votes yes or no. Only once the coordinator has collected a unanimous "yes" does it tell everyone to commit for real. If even one participant votes no, the coordinator tells everyone to abort instead, and because nothing was finalized during prepare, every participant can roll back cleanly. This gives 2PC genuine atomicity across services — strictly stronger than what a saga provides, as the next section will show.

## Why 2PC doesn't scale: the blocking problem

The weakness is specific and severe: once a participant votes yes, it has made a promise it cannot take back on its own. It's holding locks and waiting for the coordinator's final word — and if the coordinator crashes after collecting the votes but before broadcasting the decision, every participant that voted yes is now stuck. It cannot safely commit (the coordinator might have decided to abort based on some other participant's no) and it cannot safely abort (the coordinator might have already told other participants to commit). Its only correct option is to sit there, holding its locks, until the coordinator recovers and tells it what actually happened.

That's not a rare edge case to shrug off — it's the central operational cost of 2PC. Locks held indefinitely during a coordinator outage mean every other transaction waiting on those same rows or resources is blocked too, and the failure radius grows with every additional participant, since more participants means more chances for one of them (or the coordinator) to be unavailable at the critical moment. This is precisely why 2PC is rarely used across independent microservices at scale — it works fine inside a single, tightly-controlled cluster with a highly available coordinator (some distributed databases still use it internally, underneath a consensus layer that keeps the coordinator role itself from being a single point of failure), but as a pattern for coordinating arbitrary services owned by different teams, the blocking risk is usually judged not worth strict atomicity.

## Sagas: commit locally, undo explicitly if needed

The **saga pattern** takes the opposite trade. Instead of one all-or-nothing transaction spanning every service, a saga is a sequence of independent local transactions, each committing immediately and permanently on its own. If a later step fails, the saga doesn't roll back the earlier steps the way a database transaction would — nothing was ever "pending" to roll back. Instead, each step that already committed has a predefined **compensating transaction** that semantically undoes it: refund the card, release the reserved inventory, cancel the shipment.

```
Step 1: Reserve inventory      ✓ committed
Step 2: Charge card            ✓ committed
Step 3: Schedule shipment      ✗ fails

Compensate in reverse:
  Undo step 2: Refund card
  Undo step 1: Release inventory
```

No service ever blocks waiting on another service's decision — each step is final the moment it commits. The cost is that "undone" isn't the same as "never happened": a customer's card really was charged and really was refunded, which is visibly different from a transaction that simply never went through, and a system has to be designed to tolerate that visible, temporary inconsistency.

## Orchestration vs. choreography

Sagas need something deciding what happens next, and there are two common structures for that:

- **Orchestration** uses a central coordinator that explicitly tells each service what step to run next and which compensation to trigger on failure. This keeps the saga's logic in one readable place, at the cost of that orchestrator becoming a service every other step depends on.
- **Choreography** has no central coordinator at all — each service publishes an event when it finishes its step ("inventory reserved"), and the next service in line is simply listening for that event and reacts on its own, including triggering its own compensation if something downstream signals failure. This avoids a central dependency but makes the overall flow harder to see in one place, since the logic is scattered across every participating service's event handlers.

Neither structure changes the fundamental trade saga pattern makes versus 2PC: give up strict, system-wide atomicity in exchange for a design where no service ever sits blocked holding resources hostage to another service's crash.
