---
title: "Top-K Elements and the Two Heaps Pattern"
description: Tracking the k largest items with a fixed-size min-heap in O(n log k), and the max-heap/min-heap split that maintains a running median in a stream.
roadmap: datastructures-and-algorithms
node: top-k-elements-and-the-two-heaps-pattern
group: more-interview-patterns
video:
  provider: youtube
  id: "itmhHWaHupI"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 1415
level: intermediate
prerequisites: []
takeaways:
  - "To find the k largest elements among n, you never need to sort all n — keep a min-heap capped at size k, push each element, and pop the minimum whenever the heap exceeds size k; whatever survives is the top-k, in O(n log k) instead of O(n log n)."
  - "The heap is a min-heap, not a max-heap, because the element you want fast access to is the smallest member of your current top-k group — that's the one you'd evict if something bigger shows up."
  - "The 'two heaps' trick maintains a running median over a stream by splitting the data into a max-heap holding the smaller half and a min-heap holding the larger half, so the median is always at or near the two heap tops."
  - "After every insertion the two heaps are rebalanced so their sizes differ by at most one; with that invariant, the median is the top of the larger heap (odd total) or the average of both tops (even total)."
  - "Both patterns trade a full sort (O(n log n)) for a bounded-size structure, which is what makes them viable for streaming data where you can't look at the whole dataset at once and must answer a top-k or median query after every new value."
check_questions:
  - q: "Why does the top-k pattern use a min-heap of size k rather than a max-heap of size k?"
    a: "Because the operation that needs to be fast is 'is this new element bigger than my current worst member of the top-k, and if so, evict the worst one' — the worst member of a max-group is its minimum, so keeping that minimum at the root (a min-heap) makes both the comparison and the eviction O(log k) instead of requiring a scan."
  - q: "In the two-heaps median pattern, why do you insert a new number into one heap and then immediately consider moving its top to the other heap, rather than deciding up front which heap it belongs in?"
    a: "Because inserting purely by value (smaller half vs. larger half) can unbalance the heaps' sizes or let a value land in the wrong half relative to the other heap's boundary; inserting into one heap first and then shifting the extreme element across if needed is what keeps the invariant 'everything in the max-heap is ≤ everything in the min-heap' and the sizes within one of each other, no matter what order values arrive in."
  - q: "What would go wrong if you tried to solve the running-median problem by re-sorting the stream after every new element instead of using two heaps?"
    a: "It would still produce the correct answer, but each insertion would cost O(n log n) (or O(n) with an insertion-sort-like approach) instead of O(log n), since a full or partial re-sort redoes work on elements that haven't changed relative order, whereas the two-heaps approach only pays for inserting the one new element and possibly moving one element between heaps."
deeper:
  - label: "Kth Largest Element in a Stream — LeetCode 703 (min-heap walkthrough)"
    url: "https://www.youtube.com/watch?v=KRhBtCpJGgw"
related_formulas: []
---

## Two problems, one idea: don't sort more than you have to

Both halves of this pattern exist because sorting is overkill for the question actually being asked. If you only need the k largest elements, or only need the median after every new arrival, a full ordering of all n elements throws away information you never use — the relative order *within* the top-k group, or the relative order *within* each half of the data, doesn't matter. Heaps give you exactly the partial ordering each problem needs, and nothing more, which is what gets the cost down from O(n log n) to O(n log k) or O(log n) per operation.

## Fixed-size min-heap: top-k largest

Say you want the 3 largest values from the stream `[5, 1, 9, 3, 7, 2, 8]`, one value at a time, without storing everything. Keep a min-heap capped at size 3. Push 5, 1, 9 — heap holds `{1, 5, 9}` with 1 on top. Push 3: heap now has 4 elements, so pop the minimum (1); heap is `{3, 5, 9}`. Push 7: heap has 4 elements again, pop the minimum (3); heap is `{5, 7, 9}`. Push 2: it's smaller than the current minimum (5), so after pushing and popping, 2 gets immediately evicted and the heap is unchanged — `{5, 7, 9}`. Push 8: heap has 4, pop the minimum (5); heap is `{7, 8, 9}`. After the whole stream, the heap holds exactly the 3 largest values, and at every point in between, its root (the minimum of the heap) tells you the current "bar" a new element must clear to make the top-k. Each push/pop is O(log k) rather than O(log n), and you never hold more than k elements at once — the key advantage when the stream is large or unbounded.

## Two heaps: a running median

The median-of-a-stream problem needs more than "the k biggest" — it needs fast access to the boundary between the lower and upper halves of *everything seen so far*. The trick is to maintain two heaps: a max-heap called `small` holding the lower half of the values (so its largest value sits on top), and a min-heap called `large` holding the upper half (so its smallest value sits on top). The invariant kept after every insertion is twofold: every value in `small` is ≤ every value in `large`, and the two heaps' sizes never differ by more than one.

Walk it through on the stream `1, 5, 2, 8, 3`:
- Insert 1 → `small = {1}`, `large = {}`.
- Insert 5 → push 5 into `small` (now `{1,5}`, top 5), then pop that top and push it across into `large` to keep every value in `small` ≤ every value in `large`. Result: `small = {1}`, `large = {5}`. Sizes are balanced; median = average of both tops = 3.
- Insert 2 → push into `small` (`{1,2}`, top 2), pop top into `large`: `small = {1}`, `large = {2,5}`. `large` now has one more element than `small`, which is allowed (size difference ≤ 1); median = top of `large` = 2.
- Insert 8 → push into `large` directly since 8 ≥ current median region, giving `large = {2,5,8}`, `small = {1}`; sizes now differ by 2, so rebalance by popping `large`'s top (2) into `small`: `small = {1,2}`, `large = {5,8}`. Median = average of both tops = (2+5)/2 = 3.5.
- Insert 3 → compare to `small`'s top (2): 3 ≥ 2, so push into `large`: `large = {3,5,8}`, `small = {1,2}`. Sizes balanced (3 vs 2, diff of 1); median = top of `large` = 3.

Every insertion costs O(log n) for the heap push/pop, far better than re-sorting, and the median is always one or two root lookups away — O(1) to read once the heaps are balanced.
