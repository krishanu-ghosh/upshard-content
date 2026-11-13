---
title: "Cyclic sort pattern"
description: Placing each value at its "home" index in one O(n) pass exposes missing or duplicate numbers in an array with no extra space.
roadmap: datastructures-and-algorithms
node: cyclic-sort-pattern
group: more-interview-patterns
video:
  provider: youtube
  id: "JfinxytTYFQ"
  start: 0
  end: null
creator:
  name: Kunal Kushwaha
  channel_url: https://www.youtube.com/@KunalKushwaha
duration_seconds: 5732
level: intermediate
prerequisites: []
takeaways:
  - "Cyclic sort only applies when the array's n values are drawn from a known, bounded range tied to the array's length — classically 1 to n or 0 to n-1 — because that's what guarantees every value has exactly one 'home' index to be placed at."
  - "The core loop walks the array with an index i; while nums[i] does not already sit at its home index (value v belongs at index v-1 for a 1-to-n range), swap nums[i] with the element currently sitting at its home index, and only advance i once nums[i] is correctly placed or is a duplicate of the value already at its home."
  - "Each value is swapped into its home position at most once, which is why the pattern is O(n) overall even though it looks like a nested loop — the while loop inside the for loop performs at most n total swaps across the whole array, not n per iteration."
  - "After the placement pass, a second linear scan finds anomalies for free: an index i where nums[i] != i + 1 (for a 1-to-n range) means the 'home' value for that index never arrived, which is exactly the missing number; the value actually sitting there is often a duplicate that displaced it."
  - "The pattern finds missing and duplicate numbers in O(n) time and O(1) extra space, beating both a hash-set approach (O(n) time but O(n) space) and sorting first (O(n log n) time) — the array itself becomes the working space by being rearranged in place."
  - "Cyclic sort does not work on arbitrary unbounded arrays, arrays with heavy duplication beyond one or two repeats, or arrays whose values don't map cleanly to indices — recognizing the 'numbers in range 1..n (or 0..n-1), mostly or exactly matching array length' shape is the key signal to reach for this pattern."
check_questions:
  - q: "Why is the cyclic sort placement loop still O(n) overall, even though there's a while loop nested inside the outer index loop?"
    a: "Every swap performed by the inner while loop puts at least one more value into its correct home position permanently — a value is never swapped out of its home once it's placed there — so across the entire array, the total number of swaps across all iterations of the inner loop is bounded by n, making the combined cost O(n) rather than O(n^2)."
  - q: "After running the cyclic sort placement pass on an array that's supposed to contain 1..n but is actually missing one number and has one duplicate, how do you identify both the missing and the duplicate value in the follow-up scan, and why does it work?"
    a: "Scan the array checking whether nums[i] == i + 1 at each index; the first index where this fails tells you two things at once — the expected value i + 1 is the missing number (it never arrived at its home), and the value actually sitting there, nums[i], is the duplicate (it occupies two homes: its own correct slot elsewhere, and this slot where it displaced the missing number)."
  - q: "Why doesn't cyclic sort work well on an array of arbitrary large integers, like stock prices or timestamps, even though it's O(n) and in-place on bounded small-range arrays?"
    a: "Cyclic sort relies on computing an index directly from a value (value v maps to index v-1), which only makes sense when values are small integers tightly bounded by the array's own length; arbitrary large or unbounded values would map to indices far outside the array, so there's no valid 'home' position to swap toward, and the whole placement invariant breaks down."
deeper:
  - label: "Cracking the Cyclic Sort Interview Pattern to Sort an Array in Place — Coderbyte"
    url: "https://www.youtube.com/watch?v=TLiWieBQwUs"
related_formulas: []
---

## The shape that signals cyclic sort

A certain family of array problems gives you exactly `n` numbers drawn from a known,
bounded range that lines up with the array's length — "an array of size n containing
numbers from 1 to n", or "0 to n-1", possibly with one missing, one duplicated, or
both. That bounded-range-matching-length shape is the signal: it means every value has
exactly one "correct" index it should sit at, which is the precondition cyclic sort
needs. Without that bound, there's no way to compute a home index directly from a
value, and the trick doesn't apply.

## The placement pass, worked through

For a 1-to-n range, value `v` belongs at index `v - 1`. Walk the array with index `i`
from 0 to n-1. At each `i`, check: is `nums[i]` already at its home index (i.e., does
`nums[i] == i + 1`)? If yes, move on. If no, swap `nums[i]` with whatever is currently
sitting at `nums[i]`'s home index (`nums[nums[i] - 1]`), and check again at the same
`i` — don't advance until the value now at `i` is either home or a duplicate of the
value already in its home slot (in which case advance anyway, to avoid an infinite
swap loop).

Trace it on `nums = [3, 1, 5, 4, 2]` (a complete permutation of 1..5, already
correct — skip to a broken case): `nums = [3, 1, 5, 2, 2]` (5 is missing, 2 is
duplicated). Start at `i=0`: `nums[0]=3`, home index for 3 is index 2. Swap positions
0 and 2: `nums = [5, 1, 3, 2, 2]`. Still at `i=0`: `nums[0]=5`, home index 4. Swap:
`nums = [2, 1, 3, 2, 5]`. Still at `i=0`: `nums[0]=2`, home index 1. Swap:
`nums = [1, 2, 3, 2, 5]`. Still at `i=0`: `nums[0]=1`, home index 0 — correct, advance.
`i=1`: `nums[1]=2`, home index 1 — correct, advance. `i=2`: `nums[2]=3`, home index
2 — correct, advance. `i=3`: `nums[3]=2`, home index 1, but `nums[1]` is already 2 (a
duplicate) — advance anyway rather than swap forever. `i=4`: `nums[4]=5`, home index
4 — correct, advance. Final array: `[1, 2, 3, 2, 5]`.

## Reading off the answer

Now scan once more checking `nums[i] == i + 1`: index 3 fails (`nums[3] = 2`, expected
4). That single failing index tells you both facts at once — the expected value, 4, is
the **missing** number (it never got placed because 2's duplicate occupied its slot),
and the value actually sitting there, 2, is the **duplicate**. No hash set, no sort —
just the two O(n) passes and the original array used as its own scratch space.

## Why this beats the alternatives

A hash-set approach (track seen values, scan for gaps) also runs in O(n) time but costs
O(n) extra space for the set. Sorting first and scanning for a mismatch between index
and value costs O(n log n). Cyclic sort gets O(n) time and O(1) extra space by using
the array's own cells as the "set" — each swap is a constant-time write that makes
permanent progress, which is also why the total swap count across the whole pass is
bounded by n rather than unbounded.

## Where it stops applying

Cyclic sort depends entirely on being able to compute a valid in-bounds index from a
value. It breaks down for arrays of arbitrary-magnitude numbers, arrays where many
values repeat far more than once or twice, or problems where the "range" isn't tied to
the array's own length. Spotting "small integers, range matches array size, asking for
missing/duplicate/first-missing-positive" is what should trigger reaching for this
pattern over a hash set or a sort.
