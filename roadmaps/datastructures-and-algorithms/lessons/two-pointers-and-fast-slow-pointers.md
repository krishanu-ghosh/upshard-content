---
title: "Two pointers & fast/slow pointers"
description: Two coordinated index variables turn O(n^2) array problems into O(n), and the same trick at different speeds detects a linked list cycle with no extra memory.
roadmap: datastructures-and-algorithms
node: two-pointers-and-fast-slow-pointers
group: problem-solving-patterns
video:
  provider: youtube
  id: "gBTe7lFR3vc"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 643
level: intermediate
prerequisites: [dynamic-arrays-and-amortized-analysis]
takeaways:
  - "The opposite-ends two-pointer pattern starts one index at each end of a sorted array and moves whichever side can't possibly be part of a better answer, which turns an O(n^2) pair search into a single O(n) pass."
  - "Fast and slow pointers (Floyd's tortoise and hare) walk the same list at different speeds — the fast pointer taking two steps for every one the slow pointer takes — and the two meet if and only if the list contains a cycle."
  - "Both patterns replace O(n) extra memory (a hash set of seen values, or seen nodes) with O(1) extra memory, which is the entire reason they show up constantly in interviews: they're the space-optimal version of a problem you could already solve with a hash set."
  - "Finding a linked list's middle node falls out of the same fast/slow mechanism for free: when the fast pointer reaches the end, the slow pointer is sitting exactly at the midpoint, with no length pre-count needed."
  - "Once the tortoise and hare meet inside a cycle, resetting one pointer to the head and advancing both one step at a time makes them meet again exactly at the cycle's start — a second, less obvious consequence of the same invariant."
check_questions:
  - q: "Why does moving the pointer on the smaller-sum side (rather than either side arbitrarily) work in the opposite-ends two-sum pattern?"
    a: "In a sorted array, if the current pair's sum is too small, the only way to increase it is to move the left pointer right (to a larger value) — moving the right pointer left could only decrease the sum further. Moving the pointer that can't possibly help is how the pattern safely discards a whole range of candidates per step instead of checking them all."
  - q: "Why does the fast pointer moving twice as fast as the slow pointer guarantee they meet inside a cycle, rather than skipping past each other forever?"
    a: "Once both pointers are inside the cycle, the gap between them shrinks by exactly one node every step (the fast pointer gains one node of relative distance per step on a discrete, finite loop), so the gap must eventually hit zero — it can never jump over zero in a single step on a cycle of integer length."
  - q: "What would go wrong if you tried to find a linked list's middle by moving a single pointer and counting steps instead of using fast/slow pointers?"
    a: "You'd need to traverse the list once to count its total length, then traverse it again to walk to the halfway point — two full passes — whereas fast/slow pointers find the middle in a single pass, which matters when the list is read once from a stream or is expensive to re-traverse."
deeper:
  - label: "Container With Most Water — Leetcode 11 — NeetCode"
    url: "https://www.youtube.com/watch?v=UuiTKBwPgAo"
related_formulas: []
---

## Two patterns, one idea: throw away information you don't need

Both two-pointer variants exist because a naive solution checks pairs or positions it
has no real reason to check. The trick in each case is to notice an invariant that
lets you safely skip a huge chunk of the search space with every step, rather than
visiting it. That's the whole mechanism — everything else is bookkeeping.

## Opposite-ends pointers: two-sum on a sorted array

Say you have a sorted array and want to know if any two numbers sum to a target `T`.
The brute force checks every pair: O(n^2). The two-pointer version places `left` at
index 0 and `right` at the last index, and looks at `arr[left] + arr[right]`:

- If the sum equals `T`, you're done.
- If the sum is **too small**, the only way to get a larger sum is to increase the
  smaller of the two values — so move `left` one step right. Moving `right` left
  would only make the sum smaller, which is the wrong direction, so that move is
  never worth trying.
- If the sum is **too large**, symmetrically, move `right` one step left.

Each step eliminates one index permanently — either `left` only ever increases or
`right` only ever decreases, and they stop when they cross. That's n steps total,
each O(1), for O(n) overall. **Container With Most Water** is the same pattern
wearing a different costume: at each step the shorter of the two walls is the one
limiting the area, so moving the taller wall's pointer can only ever reduce the
water held, while moving the shorter wall's pointer at least has a chance of finding
something taller. Worked example: heights `[1,8,6,2,5,4,8,3,7]`. Start `left=0`
(height 1), `right=8` (height 7); area = 8 * min(1,7) = 8. Height 1 is the bottleneck,
so move `left`. Continue this way and the pointers converge on the true optimum
(`left=1, right=8`, area = 7 * 7 = 49) without ever checking most of the ~36 possible
pairs.

## Fast and slow pointers: Floyd's tortoise and hare

Now take a linked list where you can't jump to an arbitrary index — you can only
follow `next`. To detect a cycle, run two pointers from the head: `slow` advances one
node per step, `fast` advances two nodes per step. If the list is acyclic, `fast`
simply reaches `null` first and you're done — no cycle. If the list **does** loop,
here's the key invariant: once both pointers are inside the loop, the distance between
them (measured going forward around the cycle) shrinks by exactly one node every
step, because `fast` gains one extra step of progress relative to `slow` each
iteration. A shrinking non-negative integer distance must eventually hit zero — it
can't skip over zero — so the two pointers are guaranteed to land on the same node
within at most one full lap of the cycle. This is Floyd's cycle-detection algorithm,
and it uses O(1) extra space, compared to O(n) for the "store every visited node in a
hash set" alternative.

The same two-speed trick finds a list's middle without first counting its length:
advance `slow` by one and `fast` by two simultaneously; when `fast` falls off the end,
`slow` is sitting exactly at the midpoint, because `slow` has covered exactly half the
distance `fast` has covered.

## Why both matter for the patterns ahead

Opposite-ends two pointers is the direct ancestor of the sliding window technique
(the next lesson): instead of both pointers converging toward each other, a window
keeps two pointers moving in the *same* direction, one trailing the other, which is
exactly how you extend this idea from "find a pair" to "find a subarray." Fast/slow
pointers show up again anywhere you need to detect a repeating state cheaply —
cycle detection in functional-graph problems, duplicate-number tricks, and happy-number
style problems all reduce to the exact same tortoise-and-hare mechanism.
