---
title: "Prefix sums and difference arrays"
description: Precomputing running totals turns range-sum queries into O(1) lookups, and its mirror image, a difference array, turns range updates into O(1) edits.
roadmap: datastructures-and-algorithms
node: prefix-sums-and-difference-arrays
group: more-interview-patterns
video:
  provider: youtube
  id: "PhgtNY_-CiY"
  start: 0
  end: null
creator:
  name: Errichto Algorithms
  channel_url: https://www.youtube.com/@Errichto
duration_seconds: 1251
level: intermediate
prerequisites: []
takeaways:
  - "A prefix sum array prefix[i] = nums[0] + ... + nums[i-1] takes O(n) to build once, and after that any range sum nums[l..r] is prefix[r+1] - prefix[l] in O(1), because the overlapping part of the two running totals cancels out."
  - "Prefix sums only pay off when there are many queries against a fixed array; if the array changes between queries, you either rebuild in O(n) per change or switch to a structure like a Fenwick tree that supports updates."
  - "A difference array is the inverse trick: diff[i] = nums[i] - nums[i-1] lets you apply a range update (add v to every element from l to r) in O(1) by doing diff[l] += v and diff[r+1] -= v, then reconstructing the real array with one O(n) prefix-sum pass over diff at the end."
  - "The reason the difference-array update works is that adding v at index l and subtracting it at r+1 means every prefix sum computed from l through r picks up the +v exactly once, and positions after r net back to zero."
  - "2D prefix sums extend the same idea with inclusion-exclusion: sum of a submatrix is computed from four corner lookups in O(1) after an O(rows*cols) precomputation pass, because overlapping regions are added and subtracted to cancel out exactly like the 1D case."
  - "Prefix sums answer 'what's the total over a range, right now' in O(1); difference arrays answer 'apply the same change to a whole range, then tell me the final array' in O(1) per update — they solve opposite problems and are easy to confuse."
check_questions:
  - q: "Why does prefix[r+1] - prefix[l] give the sum of nums[l..r] instead of some off-by-one wrong answer?"
    a: "prefix[r+1] is the sum of everything from index 0 up to and including r, and prefix[l] is the sum of everything from 0 up to but not including l; subtracting removes exactly the part before l, leaving only indices l through r — the index shift (prefix has length n+1, with prefix[0] = 0) is what makes the subtraction clean instead of off by one."
  - q: "You need to apply 1000 different 'add v to range [l, r]' updates to an array of size n, then read the final array once. Why is a difference array dramatically faster than just looping over each range and adding v directly?"
    a: "Looping over each range directly costs O(range length) per update, which is O(n) in the worst case, so 1000 updates could cost O(1000n). A difference array turns each update into two O(1) edits (at l and r+1) regardless of range length, so all 1000 updates cost O(1000), and you pay the O(n) reconstruction cost exactly once at the end instead of once per update."
  - q: "Why doesn't the prefix-sum trick help if the array is being updated between range-sum queries?"
    a: "Prefix sums are only valid for the array state they were built from; changing a single element shifts every prefix[i] for i past that index, so you'd have to rebuild the whole prefix array in O(n) after every update, which erases the O(1)-query benefit — this is exactly the gap that structures like Fenwick trees or segment trees are built to close."
deeper:
  - label: "Prefix sum array and difference array (wcipeg wiki reference)"
    url: "https://wcipeg.com/wiki/Prefix_sum_array"
related_formulas: []
---

## The problem prefix sums solve

Given an array and many queries of the form "what's the sum of elements from index `l`
to `r`?", the naive approach re-sums the range every time: O(r - l) per query, O(n) per
query in the worst case, and O(n * q) across `q` queries. If the array never changes
between queries, you can do better by paying a one-time O(n) setup cost.

Build a `prefix` array one element longer than the input, with `prefix[0] = 0` and
`prefix[i] = prefix[i-1] + nums[i-1]` — so `prefix[i]` holds the sum of the first `i`
elements. Once built, the sum of `nums[l..r]` (inclusive) is just `prefix[r+1] - prefix[l]`.
No loop, no re-scanning — just one subtraction.

**Worked example.** Let `nums = [2, 4, 1, 5, 3]`. Then `prefix = [0, 2, 6, 7, 12, 15]`
(index `i` of `prefix` holds the sum of the first `i` elements of `nums`). To find the
sum of `nums[1..3]` (the subarray `[4, 1, 5]`, which should sum to 10): compute
`prefix[4] - prefix[1] = 12 - 2 = 10`. Correct, and done in O(1) regardless of how wide
the range is. The trick is that `prefix[4]` already contains the sum of indices 0
through 3, and `prefix[1]` contains only index 0; subtracting removes exactly the part
you didn't want.

## The inverse problem: difference arrays

Now flip the problem: instead of many range-sum *queries* on a static array, you have
many range-*update* operations — "add `v` to every element from index `l` to `r`" —
applied repeatedly, and you only need to see the final array once all updates are done.
Applying each update directly costs O(range length), which is slow if ranges are wide
and there are many updates.

A difference array flips the prefix-sum relationship around. Define
`diff[i] = nums[i] - nums[i-1]` (with `diff[0] = nums[0]`). Notice that `nums` is the
prefix sum of `diff` — reconstructing `nums` from `diff` is exactly the O(n) prefix-sum
pass from the first section. The payoff: to add `v` to the whole range `[l, r]`, you
only touch two positions: `diff[l] += v` and `diff[r+1] -= v` (skip the second step if
`r+1` is out of bounds). That's O(1) per update, no matter how wide the range is.

**Worked example.** Start with `nums = [0, 0, 0, 0, 0]`, so `diff = [0, 0, 0, 0, 0]`.
Apply "add 3 to range [1, 3]": `diff[1] += 3` and `diff[4] -= 3`, giving
`diff = [0, 3, 0, 0, -3]`. Apply "add 2 to range [0, 2]": `diff[0] += 2` and
`diff[3] -= 2`, giving `diff = [2, 3, 0, -2, -3]`. Now reconstruct `nums` by taking the
prefix sum of `diff`: `[2, 5, 5, 3, 0]`. Check it by hand: index 0 got +2 only → 2.
Index 1 got +2 and +3 → 5. Index 2 got +2 and +3 → 5. Index 3 got +3 only (the first
update's range ended at 3, the second's at 2) → 3. Index 4 got nothing → 0. It matches,
and each of the two updates cost O(1) instead of O(3).

## Why the cancellation works

Adding `v` at `diff[l]` means every prefix sum computed from `l` onward includes that
`+v` exactly once. Subtracting `v` at `diff[r+1]` cancels that `+v` back out for every
position from `r+1` onward, so the net effect is "every index in `[l, r]` gets `+v`,
nothing outside it does" — which is precisely the range update you wanted, encoded as
two edits instead of a loop.

## 2D prefix sums, briefly

The same idea extends to matrices for submatrix sum queries. Build `prefix[i][j]` as
the sum of the rectangle from `(0,0)` to `(i-1, j-1)`. A submatrix sum from
`(r1, c1)` to `(r2, c2)` is computed with inclusion-exclusion over four corner lookups:
`prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1]` — the last
term is added back because it was subtracted twice by the two subtraction terms. Setup
is O(rows * cols); each query afterward is O(1), exactly mirroring the 1D case.

## When each tool applies

Prefix sums fit a static array with many range-sum reads. Difference arrays fit many
range updates with one final read. If you need both — frequent updates *and* frequent
range-sum queries, interleaved — neither tool alone is enough, and that's the gap that
motivates Fenwick trees and segment trees later in this roadmap.
