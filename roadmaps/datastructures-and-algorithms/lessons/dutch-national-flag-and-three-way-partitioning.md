---
title: "Dutch national flag and three-way partitioning"
description: Partitioning an array into three groups in one O(n) pass with three pointers, and how it generalizes quicksort's partition step.
roadmap: datastructures-and-algorithms
node: dutch-national-flag-and-three-way-partitioning
group: more-patterns-and-tricks
video:
  provider: youtube
  id: "4xbWSRZHqac"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 947
level: intermediate
prerequisites: [two-pointers-and-fast-slow-pointers]
takeaways:
  - "The Dutch national flag algorithm partitions an array into three contiguous regions — values less than a pivot, equal to it, and greater than it — in a single O(n) pass using three pointers (low, mid, high) and O(1) extra space, with no auxiliary array."
  - "Low marks the boundary of the 'less than' region (everything before it is finalized), high marks the boundary of the 'greater than' region (everything after it is finalized), and mid is the current element being examined, scanning from the unexamined middle region outward."
  - "The three-way swap rule: if nums[mid] belongs in the low group, swap it with nums[low] and advance both low and mid (the swapped-in value at mid is already known-good); if it belongs in the high group, swap it with nums[high] and decrement high but do NOT advance mid, because the newly swapped-in value at mid hasn't been examined yet; if it already belongs in the middle group, just advance mid."
  - "The algorithm terminates when mid crosses high, at which point every index has been assigned to its correct region and the array is fully three-way partitioned without ever needing a second pass or extra storage."
  - "This is exactly the partition step of three-way quicksort: instead of partitioning around a pivot into two groups (less-than, greater-or-equal) like standard quicksort, Dutch national flag partitions into three groups in the same single pass, which is what makes three-way quicksort efficient on arrays with many duplicate keys — equal elements get grouped once and never touched again."
  - "The classic 'sort colors' problem (an array of only 0s, 1s, and 2s) is the degenerate case where the pivot value is fixed at 1: 0s are the 'low' group, 1s are the 'equal' group, and 2s are the 'high' group, so the same three-pointer logic sorts the whole array in one pass."
check_questions:
  - q: "Why does the mid pointer advance after a swap with low, but NOT advance after a swap with high?"
    a: "When mid swaps with low, the value that moves into position low is known to already belong to the 'low' group (it's the value that was at mid, already classified), so mid can safely move forward past it. When mid swaps with high, the value that moves into position mid came from the unexamined high end of the array and hasn't been classified yet, so mid must stay put and re-examine that new value on the next iteration — advancing early would let an unclassified element slip past unchecked."
  - q: "How does Dutch national flag partitioning generalize quicksort's partition step, and why does that generalization help with duplicate-heavy arrays?"
    a: "Standard quicksort partitioning splits an array into two groups around a pivot (less than, and greater-or-equal), which means equal-to-pivot elements end up mixed into one side and get re-examined in recursive calls. Dutch national flag instead carves out three groups (less than, equal to, and greater than) in the same single pass, permanently isolating all pivot-equal elements in the middle; recursion then only needs to continue on the strictly-less and strictly-greater regions, which collapses to O(n) time instead of O(n^2) on arrays with many repeated values."
  - q: "Why can this algorithm sort an array of three distinct values in a single O(n) pass with O(1) space, when comparison-based sorting is bounded at O(n log n) in general?"
    a: "The O(n log n) lower bound applies to comparison sorts over arbitrary, unbounded value ranges, where you need enough comparisons to distinguish all n! possible orderings. Here the value range is fixed to exactly three known categories, so there's no need to discover relative order between elements within the same category — each element only needs one classification decision (which of three fixed buckets it belongs to), which is why a single linear scan with constant-time swaps suffices."
deeper:
  - label: "Sort an array of 0's 1's & 2's | Intuition of Algo | Brute-Better-Optimal — take U forward"
    url: "https://www.youtube.com/watch?v=tp8JIuCXBaU"
related_formulas: []
---

## The problem the flag is named after

The algorithm is named for Edsger Dijkstra's original formulation: given balls colored
red, white, and blue (the colors of the Dutch flag) in a random arrangement, rearrange
them in place so each color forms a contiguous block, in one pass, using only swaps. The
abstraction generalizes directly to partitioning any array into "less than pivot," "equal
to pivot," and "greater than pivot" regions — the classic LeetCode "Sort Colors" problem
with values 0, 1, 2 is the special case where the pivot is fixed at 1.

## The three pointers and what they mean

Maintain three indices into the array:

- `low` — the boundary of the finalized "less than" region. Everything at index `< low`
  is confirmed to belong there.
- `high` — the boundary of the finalized "greater than" region. Everything at index
  `> high` is confirmed to belong there.
- `mid` — the current pointer scanning through the as-yet-unclassified middle zone,
  which spans `[low, high]`.

The invariant throughout the scan: `[0, low)` holds "less than" values, `[low, mid)`
holds "equal to" values, `[mid, high]` is unexamined, and `(high, n-1]` holds "greater
than" values.

## The decision rule at each step

Look at `nums[mid]` and classify it against the pivot:

1. **Belongs in "less than":** swap `nums[mid]` with `nums[low]`, then increment both
   `low` and `mid`. The value swapped into `mid`'s old slot came from the just-grown
   "equal" region (or was the low boundary itself), which is already known to be
   correctly classified, so it's safe to move past it.
2. **Belongs in "equal to":** it's already in the right place. Just increment `mid`.
3. **Belongs in "greater than":** swap `nums[mid]` with `nums[high]`, then decrement
   `high` only — do **not** advance `mid`. The value that just arrived at `mid` came
   from the unexamined high end and hasn't been classified yet; advancing `mid` here
   would silently skip examining it.

The scan ends when `mid > high` — at that point every index belongs to one of the three
finalized regions.

## Worked example: sort colors `[2, 0, 2, 1, 1, 0]`

Pivot is fixed at 1 (0 = low group, 1 = equal group, 2 = high group).
Start: `low=0, mid=0, high=5`.

- `mid=0`, `nums[0]=2` → high group. Swap `nums[0]` and `nums[5]`: `[0,0,2,1,1,2]`.
  Decrement `high` to 4. `mid` stays at 0.
- `mid=0`, `nums[0]=0` → low group. Swap `nums[0]` and `nums[0]` (no-op). Increment
  `low` to 1, `mid` to 1.
- `mid=1`, `nums[1]=0` → low group. Swap `nums[1]` and `nums[1]` (no-op). `low=2`,
  `mid=2`.
- `mid=2`, `nums[2]=2` → high group. Swap `nums[2]` and `nums[4]`: `[0,0,1,1,2,2]`.
  Decrement `high` to 3. `mid` stays at 2.
- `mid=2`, `nums[2]=1` → equal group. Increment `mid` to 3.
- `mid=3`, `nums[3]=1` → equal group. Increment `mid` to 4.
- Now `mid=4 > high=3`: stop.

Final array: `[0, 0, 1, 1, 2, 2]` — fully sorted in one pass, six element inspections,
no extra array.

## Why this is quicksort's partition step, generalized

Standard quicksort partitions around a pivot into exactly two groups: "less than or
equal" and "greater than" (or some variant). Elements equal to the pivot end up mixed
into one side, which means on an array with heavy duplication (e.g., mostly 0s and 1s)
quicksort can degrade toward O(n^2) because the "equal" elements keep getting
re-partitioned in recursive calls. Three-way quicksort uses Dutch national flag
partitioning instead: it carves out the "equal to pivot" region permanently in the same
single pass, so only the strictly-less and strictly-greater regions recurse. On
duplicate-heavy inputs this is the difference between quadratic and linear-ish behavior,
which is why three-way partitioning is the production-grade quicksort variant used when
duplicate keys are expected.
