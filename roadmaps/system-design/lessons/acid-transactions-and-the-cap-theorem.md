---
title: "ACID Transactions, BASE, and the CAP Theorem"
description: What a database transaction actually guarantees, how BASE relaxes it for scale, and why CAP forces a choice when the network splits.
roadmap: system-design
node: acid-transactions-and-the-cap-theorem
group: consistency-availability-and-consensus
video:
  provider: youtube
  id: "pomxJOFVcQs"
  start: 0
  end: null
creator:
  name: Hussein Nasser
  channel_url: https://www.youtube.com/@hnasr
duration_seconds: 2563
level: intermediate
prerequisites:
  - normalization-and-the-er-model
takeaways:
  - A transaction groups several writes into one unit that either all happen or none do — that all-or-nothing guarantee is Atomicity, the "A" in ACID.
  - Consistency means a transaction can only move the database from one valid state to another, never leaving a constraint (like a foreign key or a balance check) violated.
  - Isolation controls what one transaction can see of another's uncommitted work while both run concurrently, and Durability guarantees a committed write survives a crash.
  - BASE (Basically Available, Soft state, Eventually consistent) is ACID's looser counterpart, trading strict consistency for availability and throughput — the choice many NoSQL systems make deliberately.
  - The CAP theorem says a distributed system facing a network partition must choose between Consistency and Availability — it cannot have both at that moment, though PACELC points out there's a separate tradeoff (latency vs. consistency) even when no partition is happening at all.
check_questions:
  - q: In a transaction that moves $100 from account A to account B, which ACID property guarantees you never end up with the money deducted from A but not yet credited to B?
    a: Atomicity — the transaction's two writes are treated as a single unit, so if anything fails partway through, the whole transaction rolls back and neither write is kept.
  - q: What specifically happens to a system's guarantees during a network partition, according to CAP?
    a: It must choose between Consistency (every node sees the same, most recent data) and Availability (every request still gets a response) — it can't guarantee both while the nodes can't talk to each other.
  - q: How does BASE differ philosophically from ACID?
    a: ACID treats consistency as non-negotiable and accepts slower or less available writes to guarantee it; BASE accepts that the system may be briefly inconsistent after a write, in exchange for staying available and fast.
---

## What a transaction promises

A transaction is a group of database operations treated as a single unit. The classic example: transferring $100 from account A to account B requires two writes — debit A, credit B — and a transaction's job is to make sure the database never ends up in the state where only one of those happened.

ACID names the four guarantees a transaction makes:

- **Atomicity** — all the transaction's writes happen, or none do. If the credit to B fails after the debit from A succeeded, the whole transaction rolls back, A's debit included.
- **Consistency** — the database moves from one valid state to another. If a `balance >= 0` constraint exists, no committed transaction can leave a negative balance behind, even partway through a multi-step operation.
- **Isolation** — concurrent transactions don't see each other's uncommitted changes. Without it, a second transaction reading account A's balance mid-transfer might see the debit before the credit, and make a decision based on money that's nowhere — not yet in A, not yet in B.
- **Durability** — once a transaction commits, it survives a crash. The database has written it somewhere that power loss or a restart won't erase.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;
```

If the process crashes between the two `UPDATE`s, atomicity is what rolls the first one back instead of leaving $100 debited from A and gone.

## BASE: a different bet

ACID is expensive to guarantee across multiple machines — every node involved in a transaction has to agree before it commits, which costs time and limits throughput. Many large-scale, distributed NoSQL systems make a different bet, summarized as **BASE**:

- **Basically Available** — the system responds to every request, even if some of the data it returns is stale.
- **Soft state** — the system's state may change over time even without new input, as replicas catch up with each other.
- **Eventually consistent** — if no new writes come in, all replicas will *eventually* converge on the same value, but there's no guarantee about exactly when.

A shopping cart that occasionally shows last-minute-stale inventory, briefly, is usually an acceptable BASE tradeoff. A bank balance that's occasionally wrong is usually not — which is why banking systems lean ACID and many large consumer-facing caches and feeds lean BASE.

## CAP: a theorem about what happens when the network breaks

The CAP theorem is about a specific, unavoidable situation: a distributed system's nodes stop being able to talk to each other (a network **P**artition). When that happens, CAP says you must choose between:

- **Consistency** — every node that's still reachable returns the same, most up-to-date value, even if that means refusing some requests.
- **Availability** — every request gets *some* response, even if a node can't confirm it has the latest value.

You cannot have both during a partition — if a node can't confirm it's in sync with the others, it must either refuse to answer (favoring consistency) or answer anyway with what it has (favoring availability). This is a theorem about the partition case specifically, not an absolute statement about every database at every moment.

**PACELC** extends the idea to the much more common case where there's no partition at all: even then, a system still trades **L**atency against **C**onsistency — waiting to confirm a write has propagated to other replicas is more consistent but slower; returning immediately is faster but might read stale data a moment later. Together, CAP and PACELC are why "which database is best" has no universal answer — it depends entirely on which side of these tradeoffs a given application can afford to be on.
