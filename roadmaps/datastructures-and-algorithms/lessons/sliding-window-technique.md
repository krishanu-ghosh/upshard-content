---
title: "Sliding window technique"
description: A window that only expands and shrinks at its edges, instead of re-scanning substrings from scratch, turns an O(n^2) brute force into a single O(n) pass.
roadmap: datastructures-and-algorithms
node: sliding-window-technique
group: problem-solving-patterns
video:
  provider: youtube
  id: "wiGpQwVHdE0"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 405
level: intermediate
prerequisites: [monotonic-stacks-and-queues, deques-and-circular-buffers, two-pointers-and-fast-slow-pointers]
takeaways:
  - "A sliding window is two pointers moving in the same direction over a contiguous range, one trailing the other, as opposed to the opposite-ends two-pointer pattern where they converge toward each other."
  - "A fixed-size window (e.g. 'max sum of any k consecutive elements') adds one new element and removes one old element per step, turning an O(n*k) brute force into O(n) by never recomputing a sum from scratch."
  - "A variable-size window grows its right edge to look for more, then shrinks its left edge whenever a constraint is violated, so the left pointer also only ever moves forward — giving amortized O(n) total work even though it looks like a nested loop."
  - "The amortized-O(n) argument depends on the left pointer never moving backward: across the whole pass it advances at most n times total, no matter how many times the outer (right-pointer) loop runs, which is why the pattern beats the O(n^2) 'check every substring' brute force."
  - "A hash map or frequency array riding alongside the window (tracking counts of characters or elements currently inside it) is what makes an O(1) feasibility check possible at each step, instead of re-scanning the whole window."
check_questions:
  - q: "Why is the total work across a variable-size sliding window pass O(n) and not O(n^2), even though there's a pointer advancing inside a loop that's also advancing?"
    a: "Each of the two pointers (left and right) only ever moves forward and each can move at most n times across the entire algorithm, regardless of how the moves are interleaved — so the total number of pointer moves is bounded by 2n, not by the product of two independent loops."
  - q: "In the minimum window substring problem, why do you shrink the window from the left even though you're looking for the shortest valid window, which seems like it would call for growing, not shrinking?"
    a: "Growing the right edge is how you search for a valid window at all — once one is found, shrinking from the left is how you squeeze that valid window down to its minimum size before continuing to search for an even better one; the two moves serve different purposes (find feasibility vs. optimize size) within the same pass."
  - q: "Why doesn't the fixed-size sliding window need to re-sum all k elements every time the window moves by one position?"
    a: "The new window's sum differs from the old window's sum by exactly one addition (the newly included element) and one subtraction (the newly excluded element), so maintaining a running sum incrementally does O(1) work per shift instead of O(k) work re-adding every element in the window."
deeper:
  - label: "Minimum Window Substring — Leetcode 76 — NeetCode"
    url: "https://www.youtube.com/watch?v=jSto0O4AJbM"
related_formulas: []
---

## Why a window beats re-scanning

The brute-force approach to "find the longest substring with property P" checks every
substring: for each of the n starting positions, scan forward until P breaks, giving
O(n^2) (or worse, if checking P itself isn't O(1)). The sliding window technique
notices something the brute force throws away: when you move the window's start
forward by one position, almost everything inside the window is unchanged — you've
only added or removed a constant number of elements at the edges. Recomputing from
scratch every time is wasted work. Maintaining the window's state incrementally as
it slides is the entire optimization.

## Fixed-size windows

The simplest case: find the maximum sum of any `k` consecutive elements in an array.
Compute the sum of the first `k` elements once. Then, to slide the window one position
to the right, subtract the element leaving the window (the leftmost one) and add the
element entering it (the new rightmost one). Each slide is O(1), and there are n-k
slides, so the whole pass is O(n) instead of the O(n*k) you'd get by summing each
window from scratch.

## Variable-size windows: a worked example

**Longest substring without repeating characters** is the canonical variable-size
case. Keep a window `[left, right]` and a hash set (or frequency map) of the
characters currently inside it. Walk `right` forward one character at a time:

- If `s[right]` is not already in the window, add it, and the window has grown by one
  — update the best-length-so-far if this window is now the longest seen.
- If `s[right]` **is** already in the window (a duplicate), the window is no longer
  valid. Shrink from the left — removing `s[left]` from the set and incrementing
  `left` — until the duplicate is gone.

Trace it on `"abcabcbb"`: `right` walks a,b,c (window `"abc"`, length 3, no repeats, best
= 3). At `right` pointing to the second `a`, it's a duplicate, so shrink: remove `s[0]='a'`,
`left` becomes 1, window is now `"bca"` — no duplicate of `a` remains. Continue: next
character is `b`, also a duplicate of the `b` still in the window at index 1, so shrink
again, removing `s[1]='b'`, `left` becomes 2. This process never revisits characters
already dropped — every character is added to the window exactly once and removed at
most once, which is the source of the O(n) bound even though it feels like there's a
loop inside a loop.

**Minimum window substring** (find the shortest substring of `s` containing all
characters of `t`) runs the same mechanism with the two edge-movements swapped in
purpose: grow `right` until the window becomes *feasible* (contains everything needed),
then shrink `left` as far as possible while staying feasible, recording the window
length each time it's valid, then resume growing `right` to search for the next
feasible window. Growing searches for feasibility; shrinking optimizes size once
feasibility is achieved — both pointers still only move forward over the whole pass.

## Why this needed two-pointers and monotonic structures first

A sliding window's left and right pointers are exactly the same-direction two-pointer
movement introduced in the previous lesson, specialized to "maintain a contiguous
range" instead of "find a pair." And the sliding-window-maximum variant — where you
need the max or min of the current window at every step, not just whether the window
is feasible — needs a monotonic deque to answer that in O(1) amortized per step rather
than rescanning the window; that's precisely why monotonic stacks/queues and deques are
prerequisites here rather than afterthoughts.
