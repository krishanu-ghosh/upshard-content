---
title: "In-place linked list reversal"
description: Rewiring next pointers with a prev/curr/next triple reverses a singly linked list, or any sub-range of one, in O(n) time and O(1) space.
roadmap: datastructures-and-algorithms
node: in-place-linked-list-reversal
group: more-interview-patterns
video:
  provider: youtube
  id: "G0_I-ZF0S38"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 667
level: intermediate
prerequisites: []
takeaways:
  - "Reversing a singly linked list in place means walking it once with three pointers — prev, curr, and a saved next — and on each step rewiring curr.next to point backward to prev before advancing all three forward, never allocating a new node."
  - "Saving next = curr.next before overwriting curr.next is the step that makes the whole thing work: once you rewrite curr.next to point to prev, you've destroyed the only link to the rest of the list, so you must capture that link first."
  - "After the loop finishes, prev — not curr — is the new head of the reversed list, because curr has walked one step past the last real node (it ends at null) while prev is left sitting on the final node that was processed, which is now first."
  - "Reversing a sub-range [left, right] instead of the whole list only changes the bookkeeping at the edges: walk to the node just before left, run the same prev/curr/next reversal loop for exactly (right - left + 1) steps, then reconnect the node before the range to the new sub-range head and the sub-range's original head to whatever came after right."
  - "Reversing in groups of k is the sub-range reversal applied repeatedly: reverse the first k nodes, recursively or iteratively reverse the next k, and stitch each group's new head to the previous group's new tail, with the one subtlety that a final group shorter than k is usually left unreversed."
  - "All of these variants stay O(1) extra space because the reversal never needs a stack, array, or second list — the rewiring happens directly on the existing nodes, which is the entire point of doing it in place."
check_questions:
  - q: "Why must you save curr.next into a temporary variable before setting curr.next = prev, rather than doing the reassignment first and reading curr.next afterward?"
    a: "curr.next is the only remaining link to the rest of the unprocessed list; once you overwrite it to point backward at prev, that forward link is gone, so if you hadn't saved it first you'd have no way to advance to the next node — you'd have orphaned the remainder of the list."
  - q: "After the full-list reversal loop ends (curr has become null), why is prev the new head instead of curr?"
    a: "The loop's last useful action was processing the final real node and setting prev to point at it; curr only advances past the end to null afterward, as the loop's termination check, so curr holds no useful node reference at the end, while prev still references the actual last-processed node, which is now logically first."
  - q: "When reversing only a sub-range [left, right] of a linked list, why do you need a reference to the node just before index left, even though that node itself isn't part of the range being reversed?"
    a: "After reversing the sub-range, that node-before-left is the one whose next pointer must be updated to point at the new head of the reversed sub-range (which was originally the node at index right); without holding onto it beforehand, you'd have no way to reconnect the reversed segment back into the rest of the list, since the sub-range's original entry point no longer leads anywhere useful."
deeper:
  - label: "Reverse Nodes in K-Group — Leetcode 25 — NeetCode"
    url: "https://www.youtube.com/watch?v=1UOPsfP85V4"
related_formulas: []
---

## Why "in place" matters here

The naive way to reverse a linked list is to walk it, collect every node's value into
an array, then rebuild the list (or a new one) in reverse order. That works but costs
O(n) extra space. The in-place technique reverses the list by rewiring the existing
`next` pointers directly on the existing nodes — no new nodes, no array, O(1) extra
space beyond a few pointer variables. The mechanism is always the same three-pointer
dance: `prev`, `curr`, and a temporary `next`.

## The core loop, worked through

Start with `prev = null` and `curr = head`. At each step:

1. Save `next = curr.next` (before you destroy it).
2. Rewire: `curr.next = prev`.
3. Advance both: `prev = curr`, `curr = next`.

Trace it on `1 -> 2 -> 3 -> null`. Initial: `prev = null`, `curr = node(1)`.

- Step 1: `next = node(2)`. Rewire: `node(1).next = null`. Advance: `prev = node(1)`,
  `curr = node(2)`. List so far: `1 -> null`, with `2 -> 3 -> null` still dangling off
  to the side via `curr`.
- Step 2: `next = node(3)`. Rewire: `node(2).next = node(1)`. Advance:
  `prev = node(2)`, `curr = node(3)`.
- Step 3: `next = null`. Rewire: `node(3).next = node(2)`. Advance:
  `prev = node(3)`, `curr = null`.

Loop ends (`curr` is null). `prev` now points at `node(3)`, and following `.next`
pointers from there gives `3 -> 2 -> 1 -> null` — the fully reversed list. `prev`, not
`curr`, is the new head, because `curr`'s final advance walked it one step past the
real last node into `null`, while `prev` was left holding the last node actually
processed.

## Reversing only a sub-range

"Reverse the list between position `left` and `right`" (LeetCode 92) needs the same
loop, applied only to that slice, plus careful bookkeeping at the two boundaries.

First, walk a pointer to the node just *before* `left` — call it `beforeRange` — since
that node's `next` pointer will need to be redirected after the reversal. Then run the
exact prev/curr/next loop above, but only for `right - left + 1` steps, starting with
`curr` at the node originally at position `left`.

When it finishes, `prev` is the new head of the reversed sub-range (originally the
node at `right`). The *original* node at `left` is now the tail of the reversed
segment, and `curr` — which has walked just past the range — is sitting on whatever
comes after it. Stitch the two seams back together: `beforeRange.next = prev` connects
the part before the range to the new sub-range head, and the sub-range's original
head's `.next` is pointed at wherever `curr` landed. Nothing outside `[left, right]` is
ever touched — only the pointers at those two seams change.

## Generalizing to k-group reversal

"Reverse Nodes in k-Group" (LeetCode 25) applies sub-range reversal repeatedly: reverse
the first `k` nodes using the loop above, then treat the rest of the list as a fresh
problem and reverse its first `k` nodes, and so on, stitching each group's new head to
the previous group's new tail as you go. The one added wrinkle is a lookahead check:
before reversing a group, walk ahead `k` nodes to confirm a full group actually exists;
if fewer than `k` nodes remain, that trailing partial group is left in its original
order rather than reversed. Every group reversal is still the identical prev/curr/next
mechanism — k-group reversal is sub-range reversal applied in a loop, not a new
algorithm.

## Why this is the right tool

Any time a linked-list problem asks for a structural change (reverse all, reverse a
window, reverse groups, even some palindrome-checking and reordering problems) rather
than a value change, this three-pointer rewiring is the mechanism to reach for: it's
always O(n) time, always O(1) extra space, and the only real design decision is where
the reversal loop starts, stops, and how its boundaries get reconnected.
