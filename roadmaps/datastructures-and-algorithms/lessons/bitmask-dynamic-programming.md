---
title: "Bitmask dynamic programming"
description: Packing a subset of up to ~20 elements into one integer as DP state solves problems like the traveling salesman problem exactly, in better-than-factorial time.
roadmap: datastructures-and-algorithms
node: bitmask-dynamic-programming
group: problem-solving-patterns
video:
  provider: youtube
  id: "cY4HiiFHO1o"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 1227
level: intermediate
prerequisites: [divide-and-conquer-and-the-master-theorem, meet-in-the-middle, dynamic-programming-and-np-hard-problems]
takeaways:
  - "A bitmask DP state encodes 'which subset of up to ~20 elements has been used so far' as a single integer, where bit i being 1 means element i is included — turning a subset, which naively needs its own data structure, into a plain integer you can index an array with."
  - "The classic transition is dp[mask][i] = best way to reach a state having used exactly the elements in mask, ending at element i, built from dp[mask without i][j] for every j that could have preceded i — you remove the current bit to look up the subproblem before it was set."
  - "This brings the traveling salesman problem from O(n!) brute force down to O(n^2 * 2^n), which is still exponential but is the difference between n=12 being hopeless and n=20 being comfortably solvable."
  - "The space of masks is 2^n, which is why this pattern is specifically a small-n technique — it stops being practical well before n=25, long before array or hash-based approaches would show any strain."
  - "Checking or setting bit i of a mask with (mask >> i) & 1 and mask | (1 << i), and removing it with mask & ~(1 << i), are the handful of bitwise primitives this entire pattern is built from — the DP logic sits directly on top of them."
check_questions:
  - q: "Why does bitmask DP make the traveling salesman problem solvable for n=20 when the brute-force n! approach already fails badly past n=12?"
    a: "Brute force explores every permutation of n cities, which is O(n!) — astronomically large even for modest n (12! is about 479 million, and it only gets worse). Bitmask DP instead has one state per (subset of cities visited, current city) pair, giving O(2^n) states each with O(n) transitions, or O(n^2 * 2^n) total — for n=20 that's roughly 20^2 * 2^20, about 4*10^8, which is large but computationally feasible, versus 20! which is on the order of 10^18."
  - q: "In the TSP bitmask DP, why does the transition look up dp[mask without i][j] rather than dp[mask][j]?"
    a: "dp[mask][i] represents having already arrived at city i having visited exactly the cities in mask (which includes i). To build this state, you need to ask 'which city j did I come from, before I added i to the visited set?' — that prior state had i removed from the mask, so the transition must look up the subproblem with bit i cleared, not the current mask which already includes i."
  - q: "Why is 2^20 (about a million) an important practical ceiling for this technique, rather than the DP simply getting slower and slower as n grows past that?"
    a: "The number of distinct masks is exactly 2^n, so both the memory needed to store dp[mask][*] for every mask and the time to transition through every mask grow exponentially; past roughly n=20-22, 2^n times the per-state work exceeds what's computable in a reasonable time or fits in memory, which is a hard wall rather than a gradual slowdown — this is why the pattern is explicitly a small-n technique, unlike, say, an O(n log n) algorithm that degrades gracefully."
deeper:
  - label: "Travelling Salesman Problem source code — WilliamFiset"
    url: "https://www.youtube.com/watch?v=udEe7Cv3DqU"
related_formulas: []
---

## The idea: a subset as a number

Many DP problems need "which of these n items have I already used" as part of their
state. For n up to roughly 20, the cleanest way to represent an arbitrary subset is a
single integer: bit `i` is 1 if element `i` is included, 0 otherwise. A set of
`{0, 2, 3}` out of 5 elements becomes the binary number `01101` (bit 0, bit 2, and bit
3 set), i.e. the integer 13. This is powerful specifically because integers are cheap
to use as array indices and cheap to compare, copy, and combine — operations that
would be awkward if the subset were represented as, say, a literal set or a boolean
array. An array `dp[2^n][n]` can then hold one DP value per (subset, extra detail)
combination, with `2^n` entries addressable directly by the mask.

## The video's problem: TSP, briefly

The traveling salesman problem — shortest tour visiting every city once and returning
to the start — is the standard motivating example, and the embedded video walks its
full `dp[mask][i]` = "cheapest way to have visited exactly `mask`, ending at city `i`"
derivation, bringing O(n!) brute force down to O(n² · 2ⁿ). This roadmap's own
[dynamic programming and NP-hard problems](/roadmaps/datastructures-and-algorithms/dynamic-programming-and-np-hard-problems)
lesson derives that same Held-Karp recurrence in full if you want the
complete walkthrough — the point to take from it here is narrower: the state packs
*which cities are visited* into the mask and *where the path currently ends* as the
second dimension, and the transition peels one bit off the mask to look up the
subproblem one step earlier.

## A different shape: the assignment problem

TSP isn't the only shape bitmask DP takes, and seeing a second one is what makes the
pattern transferable rather than a memorized TSP trick. The **assignment problem**:
n workers, n tasks, a cost for every (worker, task) pair, assign each worker exactly
one task to minimize total cost. Here the state is just `dp[mask]` — no second
dimension — where `mask` is the set of tasks already assigned, and the *worker being
assigned* is implicit: it's always worker number `popcount(mask)` (the count of bits
already set), because workers are assigned in a fixed order 0, 1, 2, ....

```
dp[mask] = min over every unset bit t in mask of:
    dp[mask | (1 << t)] + cost(worker = popcount(mask), task = t)
```

**Concretely**, with 3 workers and 3 tasks: `dp[000]` asks "cheapest way to assign all
three workers given none are assigned yet." It tries giving worker 0 each task in
turn — `dp[001] + cost(0,0)`, `dp[010] + cost(0,1)`, `dp[100] + cost(0,2)` — and each of
those in turn was built the same way for worker 1, bottoming out at `dp[111]`
(all tasks taken) = 0. The answer is `dp[000]`. This is O(n · 2ⁿ) — one dimension
instead of TSP's two — because the worker index never needs its own slot in the state;
it falls straight out of how many bits are already set.

The lesson of having both examples side by side: a bitmask DP's *shape* — how many
dimensions the state needs beyond the mask itself — depends entirely on whether
"which step you're on" is implied by the mask (assignment) or needs its own tracked
value (TSP's current city, which the mask alone can't recover).

## The bitwise primitives underneath

The whole pattern rests on a handful of operations: `(mask >> i) & 1` checks whether
bit `i` is set, `mask | (1 << i)` sets bit `i`, and `mask & ~(1 << i)` clears bit `i`.
Every transition in a bitmask DP is built from these three operations plus an array
lookup — there's no deeper machinery than that.

## Why meet in the middle and divide-and-conquer come first

Bitmask DP and meet in the middle attack the same category of problem — an
exponential search space over subsets of up to ~40-ish elements — from two different
angles: meet in the middle splits the space in half and combines independently
brute-forced halves, while bitmask DP instead encodes the *whole* subset directly in
the state and reuses overlapping subproblems, trading a bigger state space (2^n) for
not needing an explicit combine step. Recognizing when each applies — meet in the
middle when the two halves are truly independent, bitmask DP when you need a running
per-element decision that depends on exactly which elements came before — depends on
having both tools available, and on the general divide-and-conquer instinct of
breaking a problem into reusable, indexable subproblems in the first place.
