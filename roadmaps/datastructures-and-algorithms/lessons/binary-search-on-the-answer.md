---
title: "Binary search on the answer"
description: When "does X work?" is monotonic in X, you can binary-search the space of possible answers directly, instead of binary-searching a sorted array.
roadmap: datastructures-and-algorithms
node: binary-search-on-the-answer
group: problem-solving-patterns
video:
  provider: youtube
  id: "U2SozAs9RzA"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 912
level: intermediate
prerequisites: [sliding-window-technique]
takeaways:
  - "Binary search on the answer doesn't search a sorted array at all — it searches the range of possible answer values (e.g. speeds from 1 to max(pile)), using a feasibility check at each midpoint instead of an equality comparison."
  - "The pattern only works when feasibility is monotonic: once a value X is 'good enough', every value larger than X (or smaller, depending on direction) must also be good enough — without that monotonic boundary, there's no single cut point for binary search to converge on."
  - "The feasibility check itself is usually an O(n) greedy or simulation pass (e.g. 'at this eating speed, how many hours does Koko need?'), which is why the total complexity is O(n log(range)) rather than O(log n) — the log factor comes from the search, not from the check."
  - "Phrases like 'minimum possible maximum', 'smallest X such that...', or 'find the minimum capacity/speed/days' are the tell that a problem wants binary search over an answer space rather than a direct formula or a search over array indices."
  - "The search range's bounds come from the problem's own structure, not from the array's sorted order — the lowest conceivable answer and the highest conceivable answer bracket the search, and the binary search narrows between them exactly as it would narrow between array indices."
check_questions:
  - q: "In the Koko Eating Bananas problem, why does binary search work on 'eating speed' even though the array of pile sizes isn't being binary-searched at all?"
    a: "The thing being searched is the space of possible eating speeds (1 to max pile size), not the array; the array is only used inside the O(n) feasibility check at each candidate speed. The feasibility property (whether a given speed finishes in time) is monotonic in speed, which is what makes binary search valid over that space."
  - q: "Why does the pattern require the feasibility function to be monotonic, and what breaks if it isn't?"
    a: "Binary search works by discarding half the remaining range based on whether the midpoint passes or fails the check, which is only safe if everything on one side of the midpoint shares the same feasibility verdict. If feasibility weren't monotonic (say, speed 4 works but speed 5 doesn't, yet speed 6 works again), discarding the half containing speed 6 based on speed 5's failure would discard a valid answer."
  - q: "Why is the overall time complexity O(n log(range)) rather than O(log n) like a normal binary search?"
    a: "A normal binary search over a sorted array does O(1) work per comparison. Here, each binary-search step calls a feasibility check that itself scans the input (e.g. computing hours needed across all piles), which costs O(n); since binary search still takes O(log(range)) steps, the total is the product of the two, O(n log(range))."
deeper:
  - label: "Capacity to Ship Packages Within D Days — Leetcode 1011 — NeetCodeIO"
    url: "https://www.youtube.com/watch?v=ER_oLmdc-nw"
related_formulas: []
---

## The shift: searching values, not positions

Ordinary binary search narrows down a position in a sorted array by comparing the
target to the midpoint's value. Binary search on the answer keeps the narrowing
mechanism but changes what's being searched: instead of array indices, the two
pointers (`lo` and `hi`) bracket a range of *candidate answers* to the actual
question being asked — a speed, a capacity, a number of days, a maximum load. At each
step you pick a candidate in the middle of that range and ask a yes/no question: "is
this candidate good enough?" If yes, the answer might be smaller (keep searching the
lower half, or record this candidate as a new best and continue); if no, the answer
must be larger. This only works because the yes/no answer is **monotonic** across the
range: all values below some threshold are infeasible, and all values at or above it
are feasible — a single, clean boundary for binary search to home in on.

## Worked example: Koko Eating Bananas

Koko has `piles` of bananas and `h` hours before the guard returns. Eating at a fixed
speed `k` bananas/hour, she needs `ceil(piles[i] / k)` hours per pile. Find the minimum
integer `k` such that she finishes all piles within `h` hours.

Brute force: try `k = 1, 2, 3, ...` until one works — potentially O(max(piles)) tries,
each O(n) to check, which is slow when pile sizes are large. Binary search on the
answer instead searches `k` directly between `lo = 1` and `hi = max(piles)`:

1. `mid = (lo + hi) // 2`. Compute total hours needed at speed `mid`: sum of
   `ceil(pile / mid)` across all piles — this is the O(n) feasibility check.
2. If total hours ≤ `h`, speed `mid` **works** — but a slower speed might also work, so
   record `mid` as a candidate answer and search the lower half: `hi = mid - 1`.
3. If total hours > `h`, speed `mid` is **too slow** — every speed below `mid` is even
   slower and therefore also fails, so search the upper half: `lo = mid + 1`.

Why is this safe? Because eating faster never increases the hours needed — feasibility
is monotonic in `k`. If speed 5 is fast enough, every speed above 5 is also fast
enough; if speed 5 is too slow, every speed below 5 is even slower. That one-directional
boundary is exactly what makes a binary search over *speeds* converge on the true
minimum in O(log(max(piles))) iterations, each costing O(n) — O(n log(max(piles)))
total, dramatically better than trying every speed one at a time.

## Recognizing the pattern

The signal is almost always linguistic before it's mathematical: "minimum capacity
such that...", "smallest number of days to...", "minimize the maximum load" are all
asking you to binary-search a range of candidate answers against a monotonic
feasibility check, not to search a sorted collection. **Capacity to Ship Packages
Within D Days** is structurally identical to Koko: binary-search the ship's capacity
between `max(weights)` (must carry at least the heaviest single package) and
`sum(weights)` (one day for everything), with the feasibility check being a greedy
simulation of how many days a given capacity requires.

## Why sliding window comes first

Binary search on the answer and the sliding window technique share a deeper kinship:
both replace an O(n^2)-or-worse brute force with a mechanism that exploits monotonic
structure to safely discard large chunks of the search space per step. Having
internalized "a window only grows and shrinks at its edges because of a monotonic
feasibility condition" makes it much more natural to see "a feasibility check that's
monotonic across a range of answers" as the same idea one level more abstract — the
object being narrowed changes from a contiguous range of an array to a numeric range
of possible answers, but the discard-half-the-space-per-step logic is identical.
