---
title: "Amortized Analysis: Aggregate, Accounting & Potential"
description: The three formal techniques for proving an amortized bound — aggregate, accounting, and potential — worked through a binary counter and a multipop stack.
roadmap: datastructures-and-algorithms
node: amortized-analysis
group: algorithm-design-techniques
video:
  provider: youtube
  id: "3MpzavN3Mco"
  start: 0
  end: null
creator:
  name: MIT OpenCourseWare
  channel_url: https://www.youtube.com/@mitocw
duration_seconds: 4553
level: intermediate
prerequisites: ["dynamic-arrays-and-amortized-analysis", "divide-and-conquer-and-the-master-theorem"]
takeaways:
  - "Amortized analysis bounds the average cost per operation over any sequence of operations, not the cost of a single worst-case operation — and the three standard techniques (aggregate, accounting, potential) are formal ways to prove that bound rigorously rather than just gesture at it."
  - "The aggregate method sums the total cost of n operations and divides by n; it proves a correct amortized bound but assigns the same average cost to every operation, even though some are genuinely cheaper than others."
  - "The accounting (banker's) method assigns each operation type its own amortized cost, overcharging cheap operations to build up prepaid 'credit' stored on data structure elements, which expensive operations later withdraw — the bound holds as long as the credit balance never goes negative."
  - "The potential method defines a single function Φ over the entire data structure's state and charges each operation's amortized cost as actual cost plus the change in potential; an expensive operation is paid for by a large drop in potential built up by preceding cheap operations."
  - "On a binary counter, incrementing flips a variable number of bits per call, but the potential method (using the count of 1-bits as Φ) proves amortized O(1) per increment, because every flip from 1→0 is paid for by a prior 0→1 that raised the potential."
  - "On a stack supporting multipop(k) (pop k elements at once), the aggregate method proves amortized O(1) per operation by noting that total pops across any sequence can never exceed total pushes, regardless of how multipop's k varies."
check_questions:
  - q: "The aggregate method and the accounting method can both prove the same O(1) amortized bound for the same data structure. What does the accounting method give you that the aggregate method doesn't?"
    a: "The aggregate method only produces one number — the average cost per operation across the whole sequence — so it can't tell you which individual operation types are expensive versus cheap; it treats push and pop as costing the same amortized amount even when they clearly don't. The accounting method assigns a distinct amortized cost per operation type (e.g., charge push $2, pop $0) and tracks credit explicitly, which lets you reason about mixed sequences of different operation types and prove the invariant (credit never negative) directly, rather than just computing an average after the fact."
  - q: "In the potential method for the binary counter, why does defining Φ as 'the number of 1-bits in the counter' make the increment operation amortized O(1), when a single increment can flip up to log n bits?"
    a: "An increment that flips k trailing 1-bits to 0 (costing k work) always also flips exactly one more 0-bit to 1 (one more unit of actual cost), for k+1 total actual cost. Flipping k ones to zeros decreases Φ by k, and flipping one zero to one increases Φ by 1, so the net change in potential is -k+1. Amortized cost = actual cost + ΔΦ = (k+1) + (1-k) = 2, a constant — the potential drop from clearing k ones exactly cancels out the k units of real work that clearing them cost, leaving only O(1) amortized regardless of how many bits that particular call happened to flip."
  - q: "Why does the aggregate method work cleanly for proving multipop is amortized O(1), without needing the more complex accounting or potential machinery?"
    a: "Multipop's cost varies per call (it pops k elements, so cost is O(k)), but there's a simple global invariant: you can never pop more elements in total than you've pushed in total, since every element popped had to be pushed first. That means across any sequence of n operations, total pop/multipop work across the whole sequence is bounded by the total number of pushes, which is at most n — so total work across all n operations is O(n), giving O(1) amortized per operation on average. Because the bound falls out directly from summing total work over the whole sequence without needing to assign different costs to different operations, the aggregate method is sufficient here; accounting or potential methods would prove the same thing but with unnecessary extra bookkeeping."
deeper:
  - label: "Amortized Analysis: Aggregate, Accounting and Potential Method (with examples)"
    url: "https://www.youtube.com/watch?v=wKla4Vqrlzw"
related_formulas: []
---

## Why "it averages out" needs a proof

The dynamic-array lesson showed informally that doubling makes append amortized O(1): expensive resizes happen, but rarely enough that their total cost, spread across all appends, averages out. That argument — sum the total cost, divide by the number of operations — is correct, but it's also the least flexible of three formal techniques for proving amortized bounds. All three answer the same question (what's the true average cost per operation over any sequence?) but differ in how much structure they expose and how well they scale to mixed sequences of different operation types.

## The aggregate method

**Idea:** compute (or bound) the total cost T(n) of *any* sequence of n operations, then the amortized cost per operation is T(n)/n. This is exactly the dynamic-array argument: total resizing cost across n appends is O(n) (a geometric series), so amortized cost per append is O(n)/n = O(1).

**Worked example — stack with multipop(k):** a stack supports push (cost 1), pop (cost 1), and multipop(k) which pops min(k, stack size) elements (cost = number popped). A naive worst-case view says multipop(k) costs O(k), which could be O(n) if the stack is large — looks bad for a single call. But look at a sequence of n operations total: every element popped (by pop or multipop) had to be pushed exactly once first, and there are at most n pushes in n operations. So total pop-work across the entire sequence is bounded by total push-work, which is at most n. Total cost across n operations is O(n), giving O(1) amortized per operation — even though any single multipop call can be expensive.

**Limitation:** the aggregate method proves one average number for the whole sequence; it can't assign different amortized costs to pushes versus multipops, which matters when operations are mixed with other costs or when you need per-operation bounds for a larger proof.

## The accounting (banker's) method

**Idea:** assign each operation type its own fixed amortized cost (possibly different from its actual cost), and bank the surplus as "credit" stored on elements of the structure. An operation's amortized cost must always be at least its actual cost *plus* whatever credit it draws from the bank — and the total credit in the system must never go negative.

**Worked example — same multipop stack:** charge push an amortized cost of 2 (actual cost 1, so 1 unit of credit is stored on the newly pushed element), and charge pop and multipop an amortized cost of 0 (their actual cost is paid entirely from the credit stored on the elements they remove). Since every element carries exactly 1 banked credit from when it was pushed, popping it (actual cost 1) is fully covered by spending that credit. Credit never goes negative because you can't pop an element that wasn't pushed (and credited) first. Total amortized cost across n operations is at most 2 per push and 0 per pop/multipop — O(n) total, O(1) amortized per operation, and this time with an explicit per-operation-type cost assignment the aggregate method couldn't give you.

## The potential method

**Idea:** define a potential function Φ over the entire data structure's state (not individual elements), with Φ(initial state) = 0 and Φ(state) ≥ 0 always. Define amortized cost of an operation as **actual cost + ΔΦ** (the change in potential it causes). If an operation's actual cost is high but it causes Φ to drop by a corresponding amount, its amortized cost stays low — the "stored potential" built up by earlier cheap operations pays for it.

**Worked example — binary counter increments:** a counter stored as a bit array supports increment, implemented the obvious way: flip trailing 1s to 0s, then flip the first 0 to a 1. An increment that clears k trailing ones costs k+1 actual work (k flips to 0, plus 1 flip to 1). Define Φ = number of 1-bits currently set. That increment changes Φ by -k (k ones cleared) +1 (one bit set) = 1-k. Amortized cost = actual cost + ΔΦ = (k+1) + (1-k) = **2**, a constant, regardless of k. Intuitively: every 1-bit that gets expensively cleared was itself cheaply created by some earlier increment (one unit of work, +1 to Φ), so the "savings account" of accumulated 1-bits always has enough balance to pay for clearing them later.

## Choosing between them

All three methods prove the same class of bound, but accounting and potential give more structure: accounting is convenient when credit naturally attaches to discrete elements (pushed items, allocated nodes), while potential is convenient when the right summary statistic of the *whole* structure's state (bit count, size, height) captures the stored-up "slack." The aggregate method is simplest when a clean total-cost argument exists and you don't need per-operation granularity — reach for it first, and move to accounting or potential when the bound involves operations trading off against each other in ways a single running total can't express.
