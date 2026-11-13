---
title: "Order Statistics & Quickselect"
description: Finding the kth smallest element without fully sorting, via Quickselect's partition-and-recurse trick, and why median-of-medians guarantees O(n).
roadmap: datastructures-and-algorithms
node: order-statistics-and-quickselect
group: advanced-sorting-and-selection
video:
  provider: youtube
  id: "XEmy13g1Qxc"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 1128
level: intermediate
prerequisites: ["non-comparison-sorts"]
takeaways:
  - "Finding the kth smallest element doesn't require sorting the whole array — sorting is O(n log n) and throws away the fact that you only need one position's worth of information, not a full ordering."
  - "Quickselect reuses quicksort's partition step: pick a pivot, partition the array around it, and the pivot's final index tells you exactly which side of the partition contains the kth element — so you only recurse into that one side, never both."
  - "Because each level only recurses into one partition instead of two, Quickselect's expected cost is O(n): the work at each level shrinks geometrically (n + n/2 + n/4 + ... = O(n)) instead of quicksort's O(n log n), which comes from recursing into both sides at every level."
  - "Quickselect's worst case is O(n²), triggered the same way as quicksort's: a pivot that's consistently the smallest or largest remaining element shrinks the problem by only one element per level instead of by half."
  - "Median-of-medians picks a pivot that's provably 'good enough' by splitting the array into groups of 5, finding each group's median in O(1), then recursively finding the median of those medians — that pivot is guaranteed to discard at least 30% of the array every time, which turns the worst case into O(n) at the cost of a larger constant factor."
  - "Randomized pivot selection (pick a uniformly random element) makes Quickselect's O(n²) worst case so unlikely in practice that most real implementations use it instead of median-of-medians, reserving the deterministic guarantee for contexts where adversarial input is a real risk."
check_questions:
  - q: "Why is Quickselect's expected running time O(n) rather than O(n log n) like quicksort, even though both use the same partition step?"
    a: "Quicksort recurses into both halves of every partition, so its recursion tree has log n levels each doing O(n) total work, for O(n log n). Quickselect only needs the kth element, so once it knows which side of the pivot that element falls on, it discards the other side entirely and recurses into just one side. With a balanced pivot, that side is about half the previous size, so the work across levels forms a geometric series n + n/2 + n/4 + ... that sums to O(n), not n log n."
  - q: "What specific kind of input drives Quickselect to its O(n²) worst case, and why doesn't that same input break median-of-medians?"
    a: "Quickselect degrades to O(n²) when the pivot is repeatedly the smallest or largest element of the remaining range — e.g. always picking the first element on already-sorted input — because each partition only shrinks the problem by one element instead of by a constant fraction. Median-of-medians avoids this because its pivot is chosen by a guaranteed procedure (median of group medians), which provably lies strictly between the extremes and discards at least 30% of the array regardless of the input's order, so no input can force it into the one-element-per-level pattern."
  - q: "Median-of-medians guarantees O(n) worst-case time, so why do most production implementations use randomized-pivot Quickselect instead?"
    a: "Median-of-medians' O(n) bound comes with a much larger constant factor — it recursively computes medians of groups of 5, which involves multiple linear passes and recursive calls before every partition — so it's noticeably slower in practice despite the better asymptotic worst case. Randomized pivot selection makes the O(n²) scenario so improbable on real or even adversarial-looking input that its much better expected-case constant factor wins out for almost every practical use; median-of-medians is reserved for situations where a guaranteed worst case matters more than average speed, such as real-time systems."
deeper:
  - label: "2.2 - Linear Time Selection (Median of Medians Algorithm)"
    url: "https://www.youtube.com/watch?v=PY7YiJQa3Lc"
related_formulas: []
---

## The question sorting over-answers

"What's the 7th smallest element in this array?" is a narrower question than "put this entire array in order" — the answer is a single value, not a full permutation. Sorting first and reading off index 6 works, but it does O(n log n) work to extract one number's worth of information. **Order statistics** is the study of finding the kth smallest (or largest) element directly, and **Quickselect** is the classic way to do it in expected linear time by borrowing quicksort's partition step but throwing away quicksort's second recursive call.

## Quickselect: partition, then look at where the pivot landed

Quickselect picks a pivot and partitions the array exactly like quicksort does: after partitioning, every element less than the pivot sits to its left, every element greater sits to its right, and the pivot itself sits at its final sorted index, call it `p`. Quicksort's next move is to recurse on both `[left, p-1]` and `[p+1, right]`. Quickselect instead compares `p` to the target rank `k` and recurses into exactly one side:

- If `p == k`, the pivot *is* the answer — stop.
- If `k < p`, the kth smallest element is in the left partition — recurse only on `[left, p-1]`.
- If `k > p`, it's in the right partition — recurse only on `[right]`, adjusting k.

Discarding the other side entirely is the whole trick. Quicksort's O(n log n) comes from doing O(n) work at each of log n levels because it visits both halves every time. Quickselect, visiting only one half, does n + n/2 + n/4 + n/8 + ... work across levels — a geometric series that sums to O(n) total, assuming each partition is reasonably balanced.

**Worked example:** find the 3rd smallest element (k=2, 0-indexed) of `[9, 3, 7, 1, 8, 2]`. Pick pivot 2 (last element); partitioning around it gives `[1, 2, 9, 7, 8, 3]` with 2 landing at index 1. Since target index 2 > 1, discard the left side including the pivot and recurse on `[9, 7, 8, 3]` looking for the element at adjusted index `2 - 1 - 1 = 0`. Pick pivot 3; partitioning gives `[3, 7, 8, 9]` with 3 at index 0 — matches the target index exactly. Answer: 3. Only one recursive call was ever made, touching 6 then 4 elements, not the full tree of calls quicksort would make.

## Why it can go quadratic — and what forces it there

Quickselect inherits quicksort's exact failure mode. If the pivot is always the minimum or maximum of the current range — the classic trap of "always pick the first element" on already-sorted (or reverse-sorted) input — each partition step only shrinks the problem by one element instead of roughly by half. The recursion depth becomes O(n), and since each level still does O(remaining size) work for the partition itself, total cost becomes O(n) + O(n-1) + O(n-2) + ... = O(n²). This is a real risk, not just a theoretical curiosity, whenever the pivot rule is predictable and the input could be adversarial or already structured (sorted, reverse-sorted, many duplicates).

## Median-of-medians: a pivot that's provably good enough

The fix is to stop picking pivots that *might* be bad and instead compute one that's *guaranteed* to be reasonably central. Median-of-medians does this recursively: split the array into groups of 5, find the median of each group by brute force (O(1) per group since the group is tiny), then recursively find the median of those group-medians using the same selection algorithm — and use that as the pivot.

The guarantee: at least half of the groups have their median ≤ the overall median-of-medians, and each of those groups contributes at least 3 of its 5 elements (the median and the two below it) that are also ≤ the pivot. Working through the arithmetic, this guarantees the pivot discards at least ~30% of the array on one side and ~30% on the other, every single time, regardless of input order. That bound — never worse than a 70/30 split — is what turns the recursion into O(n) in the worst case: solving the "find median of n/5 groups" subproblem costs T(n/5), partitioning and recursing into the larger side costs at most T(0.7n), and finding each group median is O(n) — giving the recurrence T(n) = T(n/5) + T(0.7n) + O(n), which solves to O(n) because the fractions sum to less than 1.

The price is a much larger constant factor: computing group medians and recursing twice (once for the pivot, once for the final selection) adds real overhead that a simple random pivot doesn't pay. That's why median-of-medians shows up as the deterministic fallback in library implementations (e.g. introselect) rather than the default — randomized-pivot Quickselect is faster in practice and its O(n²) case is astronomically unlikely on random pivots, but median-of-medians is there for when a guarantee, not just a probability, is required.
