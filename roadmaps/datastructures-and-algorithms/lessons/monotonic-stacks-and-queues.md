---
title: "Monotonic Stacks and Queues"
description: A stack or deque kept strictly increasing or decreasing turns O(n²) "find the next bigger/smaller element" scans into a single O(n) pass.
roadmap: datastructures-and-algorithms
node: monotonic-stacks-and-queues
group: advanced-linear-structures
video:
  provider: youtube
  id: "Dq_ObZwTY_Q"
  start: 0
  end: null
creator:
  name: AlgoMonster
  channel_url: https://www.youtube.com/@algo.monster
duration_seconds: 460
level: intermediate
prerequisites: []
takeaways:
  - "A monotonic stack is an ordinary stack with one extra rule enforced on every push: before pushing, pop off any elements that would break the stack's increasing (or decreasing) order, so the stack's contents are always sorted top to bottom."
  - "The brute-force way to find, for every element, the next greater element to its right is O(n²) — comparing each element against every element after it — while a monotonic (decreasing) stack solves the exact same problem in O(n)."
  - "The O(n) bound holds because each element is pushed onto the stack exactly once and popped at most once across the entire scan, so the total number of stack operations is bounded by 2n, not n²."
  - "Largest Rectangle in Histogram uses a monotonic increasing stack of bar indices: when a shorter bar is seen, every taller bar still on the stack has just found its right boundary, so its best possible rectangle can be finalized and popped right there."
  - "Sliding-window-maximum problems use a monotonic decreasing deque instead of a stack, because the window's left edge moves forward too — a deque lets you evict expired elements from the front in O(1) while still maintaining monotonic order from the back."
check_questions:
  - q: "Why does maintaining a monotonic stack turn an O(n²) 'next greater element' scan into O(n), when it's still doing comparisons for every element?"
    a: "The brute-force version re-scans the remaining array for every single element, which is wasted work because most of those comparisons get thrown away once a bigger element shows up further along. A monotonic stack instead holds only candidates that haven't yet found their answer, and each element only ever gets pushed once and popped once across the whole run (the pop happens the moment a qualifying bigger element appears). Since total pushes and pops are both bounded by n, the total work across the entire array is O(n), not O(n) work repeated n times."
  - q: "In the Largest Rectangle in Histogram problem, why does popping a bar off the monotonic stack happen exactly when a shorter bar is encountered?"
    a: "A bar's rectangle can only extend as far right as the first shorter bar that follows it, because a shorter bar caps how tall any rectangle spanning past it can be. As long as the stack stays increasing, every bar on it still has an open-ended right boundary — its limit hasn't been found yet. The moment a new, shorter bar arrives, every taller bar still on the stack has just had its right boundary determined for the first time, so each can be popped and have its maximum rectangle area computed immediately, using the new bar's index as the right edge and whatever's left on the stack as the left edge."
  - q: "Why does sliding-window-maximum need a deque instead of a plain monotonic stack?"
    a: "A stack only ever adds and removes from one end, which works for 'next greater to the right' because old elements never need to leave except by being popped when they're beaten. But a sliding window also has a left boundary that moves forward, which can expire the front of the structure — an element still at the front might now be outside the window even though nothing bigger has come along yet. A deque supports O(1) removal from both ends, so new elements get added (and smaller ones evicted) from the back exactly like a monotonic stack, while expired elements that fell out of the window get evicted from the front — something a plain stack has no mechanism for."
deeper:
  - label: "Largest rectangle in Histogram | Leetcode #84 (Techdose)"
    url: "https://www.youtube.com/watch?v=vcv3REtIvEo"
related_formulas: []
---

## The pattern: only keep candidates that could still win

A **monotonic stack** is a regular stack with one invariant bolted on: its elements are always sorted, either strictly increasing or strictly decreasing from bottom to top. That invariant is maintained by changing what happens on push — before pushing a new element, you pop off every element at the top that would violate the order, and critically, *that popping is where the useful work happens*. Each popped element gets "resolved" against the new element in some problem-specific way (its answer gets recorded) before being discarded.

The pattern shows up whenever a problem asks, for every element, "what is the nearest element to the left or right that is bigger (or smaller) than me?" The brute-force answer scans forward (or backward) from every single element until it finds a qualifying one, which costs O(n) per element and O(n²) overall. A monotonic stack answers the same question for every element in a single left-to-right pass.

## Worked example: next greater element

Take the array `[2, 1, 5, 3, 4]` and find, for each element, the next element to its right that's strictly greater than it (or "none" if there isn't one).

Keep a **monotonic decreasing** stack of *indices*, scanning left to right:

- `i=0` (value 2): stack is empty, push index 0. Stack: `[0]`.
- `i=1` (value 1): top of stack is index 0 (value 2), which is already bigger than 1, so the decreasing order holds — push. Stack: `[0, 1]`.
- `i=2` (value 5): top is index 1 (value 1). 5 > 1, so pop index 1 — its next greater element is 5. Top is now index 0 (value 2). 5 > 2, so pop index 0 too — its next greater element is also 5. Stack is now empty, push index 2. Stack: `[2]`.
- `i=3` (value 3): top is index 2 (value 5), which is bigger than 3, order holds — push. Stack: `[2, 3]`.
- `i=4` (value 4): top is index 3 (value 3). 4 > 3, pop index 3 — its next greater element is 4. Top is now index 2 (value 5), which is bigger than 4, order holds — push index 4. Stack: `[2, 4]`.
- End of array: anything left on the stack (indices 2 and 4, values 5 and 4) has no next greater element.

Result: `2→5, 1→5, 5→none, 3→4, 4→none`. Every index was pushed exactly once and popped at most once, for a total of at most 2n stack operations across the whole scan — that's the source of the O(n) bound, in contrast to the O(n²) of re-scanning from every position.

## Largest Rectangle in Histogram

Given bar heights `[2, 1, 5, 6, 2, 3]`, the goal is the largest rectangular area using contiguous bars, where the rectangle's height is capped by the shortest bar it spans. The key insight: for each bar, the largest rectangle *with that bar as the shortest one* extends left and right until it hits a shorter bar on either side — so the problem reduces to finding, for every bar, its nearest shorter bar on the left and on the right.

A **monotonic increasing** stack of indices handles both in one pass: scan left to right, and whenever the incoming bar is shorter than the bar at the top of the stack, that's exactly the signal that the top bar's right boundary has just been found (the current index), while whatever remains on the stack below it after popping gives the left boundary. Pop it, compute its area using those two boundaries, and continue — the same "pop resolves an answer" mechanic as next-greater-element, just computing a different quantity at pop time.

## Sliding window maximum: when a stack isn't enough

Finding the maximum in every size-k window of an array adds a complication a stack can't handle alone: the window's *left* edge moves forward too, so an element can become irrelevant not because something bigger beat it, but because it aged out of the window. The fix is a **monotonic decreasing deque** of indices: new elements are pushed from the back exactly like a monotonic stack (popping anything smaller first), but before reading the current maximum, the front of the deque is also checked and evicted if its index has fallen outside the window. Both the back-eviction and front-eviction happen at most once per element, keeping the whole algorithm O(n) — the same amortized argument as the stack case, just with eviction happening from two ends instead of one.
