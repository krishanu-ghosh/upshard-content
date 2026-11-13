---
title: "Merge intervals pattern"
description: Sorting intervals by start time turns overlapping ranges into a single left-to-right sweep that merges, inserts, or counts overlaps.
roadmap: datastructures-and-algorithms
node: merge-intervals-pattern
group: more-interview-patterns
video:
  provider: youtube
  id: "44H3cEC2fFM"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 614
level: intermediate
prerequisites: []
takeaways:
  - "Sorting intervals by start time is what makes a single linear sweep sufficient: once sorted, any interval that overlaps the one you're building can only be the next one in order, so you never need to look backward or re-scan."
  - "Merging works by keeping a 'current merged interval' and comparing each next interval's start to the current interval's end: if the next start is less than or equal to the current end, they overlap and you extend the current end to the max of the two ends; otherwise the current interval is finished and the next one starts a new group."
  - "Insert interval is the same sweep with one twist: intervals strictly before the new one are copied as-is, intervals strictly after are copied as-is, and everything in between that overlaps the new interval gets merged into it before it's placed — so you never need to re-sort after inserting."
  - "Meeting rooms / minimum rooms needed is a variant that counts overlap depth rather than merging ranges: sort start times and end times separately, then sweep both pointers, incrementing a room counter on a start and decrementing on an end, tracking the maximum concurrent count."
  - "Interval intersection (finding the overlap between two already-sorted interval lists) uses two pointers instead of one, advancing whichever interval ends first after recording the overlap — a direct extension of the start-time-sort insight to two lists instead of one."
  - "The whole family's time complexity is dominated by the sort, O(n log n); the sweep itself is always a single O(n) pass once intervals are ordered, which is why 'sort by start, then sweep' is the first thing to try on any interval problem."
check_questions:
  - q: "Why does sorting intervals by start time guarantee that a single forward sweep is enough to merge all overlaps, instead of needing to re-check earlier intervals?"
    a: "Once sorted by start, if interval i doesn't overlap the current merged interval (its start is after the current end), no interval after i can overlap it either, since every later interval's start is even larger — overlaps can only ever come from the immediately following intervals in sorted order, never from ones already passed, so nothing earlier ever needs revisiting."
  - q: "In the minimum-meeting-rooms problem, why do you sort start times and end times into two separate arrays instead of sorting the list of (start, end) interval pairs as units?"
    a: "You need to know, at any moment, how many meetings have started but not yet ended — that requires comparing the next start against the next end across the whole timeline, not just within one meeting's pair, so separating starts and ends lets you merge-walk two independently sorted timelines and track concurrent count directly, which a single sort of interval objects can't give you as cleanly."
  - q: "When inserting a new interval into an already-merged, sorted list, why can intervals be classified into 'strictly before', 'overlapping', and 'strictly after' groups in a single pass without re-sorting?"
    a: "Because the existing list is already sorted and non-overlapping, once you hit an interval that starts after the new interval's end, every subsequent interval also starts later (sorted order) and so cannot overlap either — meaning the 'before', 'overlapping', and 'after' groups are each a contiguous run in the original order, visitable with one pointer sweeping forward exactly once."
deeper:
  - label: "Merging Intervals — GeeksforGeeks"
    url: "https://www.geeksforgeeks.org/dsa/merging-intervals/"
related_formulas: []
---

## Why sorting first is the whole trick

Given a list of intervals like `[[1,3],[2,6],[8,10],[15,18]]`, "merge all overlapping
intervals" looks like it might need comparing every pair — an O(n^2) scan. It doesn't,
because of one observation: if you sort the intervals by their **start** value, any
interval that could possibly overlap the one you're currently building can only appear
immediately next in that sorted order. Nothing earlier needs rechecking, and nothing
far ahead can sneak in and overlap retroactively. That turns an all-pairs comparison
into a single forward sweep.

## The merge sweep, worked through

Sort by start: `[[1,3],[2,6],[8,10],[15,18]]` is already sorted here. Initialize the
result with the first interval, `current = [1,3]`. Walk the rest:

- Next is `[2,6]`. Its start (2) is `<=` `current`'s end (3), so they overlap. Extend:
  `current = [1, max(3,6)] = [1,6]`.
- Next is `[8,10]`. Its start (8) is `>` `current`'s end (6) — no overlap. Push
  `[1,6]` to the result, and start a new `current = [8,10]`.
- Next is `[15,18]`. Start (15) `>` `current`'s end (10) — no overlap. Push `[8,10]`,
  new `current = [15,18]`.
- End of list: push the final `current = [15,18]`.

Result: `[[1,6],[8,10],[15,18]]`. Every decision was a single comparison against the
interval currently being built — no backtracking, no re-scanning.

## Insert interval: the same sweep, one new interval

Given an already-merged, sorted interval list and one new interval to add, you don't
need to append and re-sort (which would cost O(n log n) again). Instead, sweep once:
copy every interval that ends strictly before the new interval's start unchanged; then
merge every interval that overlaps the new interval into it (growing the new interval's
start to the min and its end to the max as you go); then copy every remaining interval
unchanged. This works because sortedness guarantees those three groups are each a
contiguous block in the original order — you never have to jump back once you've moved
past the overlapping block.

## Meeting rooms: counting overlap depth, not merging ranges

"What's the minimum number of rooms needed to hold all these meetings?" is a different
question — not "what's the union of these ranges" but "what's the maximum number of
ranges active at the same instant." The trick: sort all start times and all end times
into two separate sorted arrays, then walk them with two pointers. Each time the next
start time is earlier than the next end time, a new room is needed (increment a
counter, advance the start pointer); otherwise a meeting has freed up a room (decrement,
advance the end pointer). The running maximum of the counter is the answer. On
`[[0,30],[5,10],[15,20]]`: starts `[0,5,15]`, ends `[10,20,30]`. Start 0 < end 10 →
rooms=1. Start 5 < end 10 → rooms=2. Now the next start (15) is greater than the next
end (10), so a meeting has freed up a room first: rooms=1, advance the end pointer to
20. Re-compare: start 15 < end 20 → rooms=2 again, and the starts are exhausted. Peak
was 2 rooms.

## Interval intersection: two sorted lists, two pointers

Given two already-sorted, non-overlapping interval lists, find all pairwise overlaps
between them. Instead of one pointer, use two — one per list. At each step, compute the
overlap between the current interval from each list (if `max(startA, startB) <=
min(endA, endB)`, record `[max(startA,startB), min(endA,endB)]`), then advance whichever
interval ends first, since it can't possibly overlap anything further in the other list.
This is the direct two-list generalization of the single-list sweep.

## The common thread

Every variant here — merge, insert, count rooms, intersect — relies on the same fact:
once intervals are ordered by their start (or, for meeting rooms, once starts and ends
are each ordered independently), overlap can only be checked against *neighbors* in
that order, never against something already passed. That's what collapses an apparent
O(n^2) all-pairs problem into an O(n log n) sort plus an O(n) sweep.
