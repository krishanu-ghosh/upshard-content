---
title: "Kadane's algorithm and maximum subarray"
description: Finding the maximum-sum contiguous subarray in O(n) by framing each position as a tiny 1D dynamic-programming decision.
roadmap: datastructures-and-algorithms
node: kadanes-algorithm-and-maximum-subarray
group: more-patterns-and-tricks
video:
  provider: youtube
  id: "5WZl3MMT0Eg"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 483
level: intermediate
prerequisites: [dynamic-programming-and-np-hard-problems]
takeaways:
  - "Kadane's algorithm is a 1D dynamic program where the state at each index i is 'the maximum sum of a subarray that ends exactly at i' — not the maximum sum seen anywhere so far, which is a separate running answer tracked alongside it."
  - "The recurrence at each step is best_ending_here = max(nums[i], best_ending_here + nums[i]) — either you extend the previous best-ending-here subarray by one element, or you abandon it and start a brand-new subarray at i, whichever gives a larger sum."
  - "Starting fresh at i is only better than extending when the running sum has gone negative, because a negative prefix can only drag down everything appended after it — a non-negative running sum is never worth discarding."
  - "Because the recurrence only ever needs the previous state, not a full table, the DP collapses to O(1) extra space — two scalars, best_ending_here and best_overall — instead of the O(n) array a naive DP formulation would suggest."
  - "The algorithm runs in a single O(n) pass with no backtracking, which is what makes it strictly better than the O(n^2) brute force of checking every (start, end) pair or the O(n log n) divide-and-conquer alternative."
  - "The same 'extend-or-restart' shape reappears across many problems (maximum product subarray, best time to buy/sell stock, longest increasing run) — recognizing a 1D scan where the decision at each step is 'extend what I was building or cut my losses and restart' is the transferable signal, not just the maximum-subarray problem itself."
check_questions:
  - q: "Why is Kadane's algorithm considered a dynamic programming technique rather than just a clever loop, and what exactly is the DP state?"
    a: "It is a DP because it defines a state, best_ending_here(i) = the max sum of a subarray ending exactly at index i, and computes it from a recurrence on the previous state, best_ending_here(i) = max(nums[i], best_ending_here(i-1) + nums[i]); it's a 1D DP whose state collapses to a single rolling scalar since each step only depends on the immediately prior one, so no table is ever materialized."
  - q: "If best_ending_here becomes negative at some index, why does the algorithm effectively 'restart' instead of continuing to extend?"
    a: "A negative best_ending_here means the subarray ending at the current index is dragging the total down below what a fresh start would give; since max(nums[i], best_ending_here + nums[i]) picks nums[i] alone whenever best_ending_here is negative, the recurrence naturally discards the negative prefix and begins a new candidate subarray at the current element, without needing a separate explicit 'reset' branch."
  - q: "Why does Kadane's algorithm need two tracked values, not just one, and what would go wrong if you only tracked best_ending_here?"
    a: "best_ending_here can legitimately drop (e.g., after a large negative number resets it) even while a better subarray was found earlier in the scan; without a separate best_overall that takes the max across all positions, the final value of best_ending_here only reflects the subarray ending at the very last index, which may not be the true global maximum."
deeper:
  - label: "Kadane's Algorithm | Maximum Subarray Sum | Finding and Printing — take U forward"
    url: "https://www.youtube.com/watch?v=AHZpyENo7k4"
related_formulas: []
---

## The brute force, and why it's wasteful

The maximum subarray problem asks: given an array of integers (positive and negative),
find the contiguous slice with the largest sum. The obvious approach checks every pair
of start and end indices, summing each candidate slice — O(n^2) time, or O(n^3) if you
re-sum from scratch instead of accumulating. The waste is that most of that work is
redundant: the sum of `nums[i..j]` is just the sum of `nums[i..j-1]` plus `nums[j]`, so
there's no need to recompute it from zero every time.

## Reframing it as a 1D DP

Define `best_ending_here(i)` as the maximum sum of any subarray that ends exactly at
index `i` (it must include `nums[i]`). This is a genuine DP state: it has a recurrence
that depends only on the state one step back.

At index `i`, a subarray ending at `i` either (a) is just `nums[i]` by itself, or (b) is
some subarray ending at `i-1` extended by one more element. Since you want the best
possible sum, you take whichever option is larger:

```
best_ending_here[i] = max(nums[i], best_ending_here[i-1] + nums[i])
```

This is the entire insight: **extend the previous best-ending-here subarray, or abandon
it and start fresh at the current element** — pick whichever yields a bigger sum. A
second variable, `best_overall`, tracks the maximum `best_ending_here` value seen across
all indices, since the answer isn't necessarily the subarray ending at the last
position.

Because `best_ending_here[i]` only ever reads `best_ending_here[i-1]`, you never need an
array — two scalars updated in a single pass suffice. That's why Kadane's algorithm is
O(n) time and O(1) space, even though it's "really" a dynamic program under the hood.

## Worked example

Take `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`. Initialize `best_ending_here = nums[0] =
-2` and `best_overall = -2`.

- `i=1`, `nums[1]=1`: extend gives `-2+1=-1`; fresh gives `1`. Max is `1`.
  `best_ending_here=1`. `best_overall=max(-2,1)=1`.
- `i=2`, `nums[2]=-3`: extend gives `1+(-3)=-2`; fresh gives `-3`. Max is `-2`.
  `best_ending_here=-2`. `best_overall` stays `1`.
- `i=3`, `nums[3]=4`: extend gives `-2+4=2`; fresh gives `4`. Max is `4` — the negative
  prefix wasn't worth carrying, so this is effectively a restart.
  `best_ending_here=4`. `best_overall=max(1,4)=4`.
- `i=4`, `nums[4]=-1`: extend gives `4+(-1)=3`; fresh gives `-1`. Max is `3`.
  `best_ending_here=3`. `best_overall` stays `4`.
- `i=5`, `nums[5]=2`: extend gives `3+2=5`; fresh gives `2`. Max is `5`.
  `best_ending_here=5`. `best_overall=max(4,5)=5`.
- `i=6`, `nums[6]=1`: extend gives `5+1=6`. `best_ending_here=6`.
  `best_overall=max(5,6)=6`.
- `i=7`, `nums[7]=-5`: extend gives `6+(-5)=1`; fresh gives `-5`. Max is `1`.
  `best_ending_here=1`. `best_overall` stays `6`.
- `i=8`, `nums[8]=4`: extend gives `1+4=5`; fresh gives `4`. Max is `5`.
  `best_ending_here=5`. `best_overall` stays `6`.

Final answer: `6`, from the subarray `[4, -1, 2, 1]`. Notice the "restart" moment at
`i=3`: the prefix `[-2, 1, -3]` summed to `-4`, strictly worse than starting over at `4`
alone — which is exactly why `max(nums[i], best_ending_here + nums[i])` discards it.

## Why this generalizes

The "extend or restart" shape isn't unique to sums. The same recurrence structure
(carry a running best that resets whenever continuing it stops being worthwhile) appears
in maximum product subarray (with a twist for sign flips), best-time-to-buy-and-sell
stock variants, and longest consecutive runs satisfying some property. Spotting "I'm
scanning once, and at each step I either keep building on what I had or cut my losses"
is the signal that a Kadane-style 1D DP with O(1) state applies.
