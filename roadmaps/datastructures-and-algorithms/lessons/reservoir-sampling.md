---
title: "Reservoir sampling"
description: Selecting a uniform random sample of size k from a stream of unknown length in one pass and O(k) space.
roadmap: datastructures-and-algorithms
node: reservoir-sampling
group: more-patterns-and-tricks
video:
  provider: youtube
  id: "DWZqBN9efGg"
  start: 0
  end: null
creator:
  name: Coding Interview Prep
  channel_url: https://www.youtube.com/@CodingInterviewPrep
duration_seconds: 761
level: intermediate
prerequisites: [probability-and-combinatorics-for-algorithms]
takeaways:
  - "Reservoir sampling selects a uniform random sample of size k from a stream whose total length is unknown or too large to fit in memory, using only one pass and O(k) space — it never needs to know n in advance or store the whole stream."
  - "For k=1: keep a running 'chosen' element, and when the i-th item arrives (1-indexed), replace the chosen element with it with probability 1/i; this guarantees every item seen so far has exactly a 1/i chance of being the one currently held, which generalizes to every item ending with exactly a 1/n chance once the stream of length n is exhausted."
  - "The replacement-probability proof for k=1 is an induction: assume after i-1 items each has probability 1/(i-1) of being held; the i-th item is kept with probability 1/i, and every previously-held item survives only if it is NOT replaced, which happens with probability 1 - 1/i = (i-1)/i; multiplying the prior 1/(i-1) chance by (i-1)/i gives exactly 1/i — so the invariant holds at every step by induction."
  - "The general k-sample version (Algorithm R) fills the reservoir with the first k items unconditionally, then for each later i-th item (i > k), generates a random integer j in [1, i]; if j <= k, the item replaces reservoir slot j, otherwise it's discarded — this is the direct generalization of the k=1 replace-with-probability-1/i rule to a multi-slot reservoir."
  - "Each item's overall probability of ending up in the final k-sample works out to exactly k/n regardless of when it arrived in the stream, which is the uniformity guarantee reservoir sampling provides — early items and late items are equally likely to survive to the final sample."
  - "Reservoir sampling is the standard answer to streaming interview questions like 'pick a uniformly random line from a file of unknown size' or 'return a random node from a linked list' where a second pass to count total length first is disallowed or impossible."
check_questions:
  - q: "In the k=1 version, why does replacing the held element with probability 1/i (rather than some other probability) at the i-th item guarantee a uniform 1/n chance at the end?"
    a: "By induction: assume every item among the first i-1 currently has probability exactly 1/(i-1) of being held. The i-th item becomes the held element with probability 1/i by construction. Any previously-held item survives only if the i-th item does NOT replace it, which happens with probability 1 - 1/i = (i-1)/i. Multiplying its prior probability 1/(i-1) by the survival probability (i-1)/i gives (i-1)/((i-1)*i) = 1/i — exactly matching the new item's probability. So after processing all n items, every item ends with probability 1/n, by induction from the base case where the first item starts at probability 1/1 = 1."
  - q: "Why can't you just pick a uniformly random index in [1, n] directly and grab that item, instead of using reservoir sampling?"
    a: "That approach requires knowing n up front, which either means making two passes over the stream (one to count, one to extract) or buffering the whole stream in memory to find its length — exactly what reservoir sampling is designed to avoid. Reservoir sampling achieves the same uniform 1/n probability per item in a single pass using O(1) (or O(k)) space, without ever needing to know the stream's total length in advance."
  - q: "In the general k-sample Algorithm R, why does the i-th item (for i > k) only have a k/i chance of entering the reservoir at all, and why is that the correct rate?"
    a: "The algorithm draws a random integer j uniformly from [1, i], and the item only enters the reservoir if j falls within the first k slots (j <= k) out of i possible values — which happens with probability k/i. This rate is correct because it exactly balances the 'new item displaces something' probability against the 'existing items must survive' probability across all future steps, which is what the inductive proof (the k=1 case generalized per-slot) shows converges to a uniform k/n chance for every item once the full stream of length n has been processed."
deeper:
  - label: "Reservoir Sampling — an interactive visual explainer (Sam Rose)"
    url: "https://samwho.dev/reservoir-sampling/"
related_formulas: []
---

## The problem: sampling without knowing how much there is

Imagine reading lines from a file, or events from a live stream, and you're asked to
return one uniformly random line — but you don't know how many lines there are until
the stream ends, and the stream might be too large to hold in memory. You also only get
to look at each item once. Buffering everything and picking a random index afterward
defeats the purpose; you need an algorithm that makes a decision about each item as it
arrives, using a fixed, small amount of memory, and still ends up with a perfectly
uniform result.

## The k=1 case: one running choice

Keep a single variable, `chosen`, initialized to the first item. As the stream
continues, when the `i`-th item arrives (counting from 1), replace `chosen` with it with
probability `1/i`. Concretely, draw a random integer from `1` to `i`; if it equals `1`
(which happens with probability `1/i`), replace; otherwise keep the current `chosen`.

Why does this end up exactly uniform? Prove it by induction. **Base case:** after seeing
1 item, it's held with probability `1/1 = 1` — trivially uniform over a stream of length
1. **Inductive step:** assume that after `i-1` items, every item seen so far has
probability exactly `1/(i-1)` of currently being `chosen`. When the `i`-th item arrives:
it becomes `chosen` with probability `1/i` (true uniformly by construction). Any item
that was previously `chosen` remains `chosen` only if the new item does *not* replace it
— which happens with probability `1 - 1/i = (i-1)/i`. So its probability becomes
`1/(i-1) * (i-1)/i = 1/i`. Every item — new or old — now has probability exactly `1/i`
of being held. This holds at every step, so after the full stream of `n` items, every
item has exactly a `1/n` chance of being the final answer.

## Worked example, k=1

Stream: `[A, B, C, D]`. Start: `chosen = A` (probability 1).

- `i=2`, item `B`: replace with probability `1/2`. Say it replaces. Now `chosen = B`;
  `A`'s survival required *not* replacing (prob `1/2`), so `A`'s running probability is
  `1 * 1/2 = 1/2` — matches `B`'s `1/2`. Tied, as expected for 2 items.
- `i=3`, item `C`: replace with probability `1/3`. Whichever of `A`/`B` was held before
  survives with probability `2/3`, giving it `1/2 * 2/3 = 1/3`. `C` itself holds `1/3`.
  All three now sit at `1/3`.
- `i=4`, item `D`: replace with probability `1/4`. The previously held item survives
  with probability `3/4`, giving `1/3 * 3/4 = 1/4`. `D` holds `1/4`. All four items now
  have exactly `1/4` — uniform over the full stream, confirmed.

## Generalizing to k items: Algorithm R

For a sample of size `k`, first fill the reservoir unconditionally with the first `k`
items. For every subsequent item at position `i` (`i > k`), draw a random integer `j`
uniformly from `[1, i]`. If `j <= k`, the new item replaces whatever currently sits in
reservoir slot `j`; otherwise, it's discarded. This is the direct multi-slot
generalization of the k=1 rule: instead of a single "replace with probability `1/i`"
decision, there are `k` "slots" that collectively get replaced with probability `k/i`,
and each specific slot is equally likely to be the one touched. The same inductive
argument (now tracking each slot's occupancy probability rather than a single `chosen`
variable) shows every item in the stream ends with exactly a `k/n` chance of surviving
into the final reservoir of size `k`, using only O(k) space and one pass — no matter how
large or unbounded the stream turns out to be.
