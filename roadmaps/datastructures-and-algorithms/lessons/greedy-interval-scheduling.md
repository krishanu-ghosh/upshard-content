---
title: "Greedy interval scheduling"
description: Sorting activities by end time and greedily taking the next compatible one provably maximizes non-overlapping intervals; sorting by start time does not.
roadmap: datastructures-and-algorithms
node: greedy-interval-scheduling
group: more-interview-patterns
video:
  provider: youtube
  id: "nONCGxWoUfM"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 772
level: intermediate
prerequisites: [merge-intervals-pattern, greedy-algorithms]
takeaways:
  - "The activity selection / interval scheduling problem asks for the maximum number of non-overlapping intervals you can keep from a set, not for merging overlaps together — it's a selection problem, not a union problem, even though it reuses the sort-the-intervals setup."
  - "The greedy rule is: sort all intervals by end time (ascending), then repeatedly take the next interval whose start is at or after the end of the last interval you kept, skipping any that overlap it."
  - "Sorting by start time instead of end time does not work: a long interval that starts earliest can block out many short, non-overlapping intervals that would have let you keep more total intervals — start-time-first greedily commits to intervals that look good locally but are globally wasteful."
  - "The proof that end-time-first greedy is optimal is an exchange argument: for any optimal solution, you can swap its first-chosen interval for the one with the globally earliest end time without decreasing the solution's size, because the earliest-ending interval is 'least restrictive' on what can follow it — repeating this swap shows the greedy choice is always at least as good."
  - "This pattern directly extends the merge-intervals sort-and-sweep setup to a different objective — instead of merging adjacent overlaps into one interval, you're discarding overlapping intervals entirely to keep a maximal independent set, which is why merge-intervals-pattern is the prerequisite here."
  - "On LeetCode this is 'Non-overlapping Intervals': instead of asking directly for the max number you can keep, it asks for the minimum number to remove — which is just (total intervals) minus (max number keepable by the end-time-greedy rule), so the same greedy algorithm answers both phrasings."
check_questions:
  - q: "Why does sorting by start time and greedily keeping the earliest-starting compatible interval fail to always maximize the number of non-overlapping intervals, when sorting by end time works?"
    a: "An interval that starts earliest can still be very long, blocking out a stretch of time where several shorter, mutually non-overlapping intervals could otherwise have been fit in — greedily committing to it based only on its start time ignores how much future opportunity it forecloses, whereas sorting by end time always picks whichever compatible interval frees up time soonest, preserving the most room for everything that comes after."
  - q: "What is the core idea of the exchange argument that proves sorting by end time and greedily selecting is optimal for interval scheduling?"
    a: "Take any optimal solution and look at its first-chosen interval; the interval with the globally earliest end time ends at or before that first interval ends, so swapping it in instead cannot make fewer later intervals compatible than before (anything compatible with the original first choice is still compatible with an earlier-ending replacement) — so the swap never shrinks the solution, meaning the earliest-end-time interval could always have been chosen without loss, which is exactly the greedy rule, applied repeatedly down the rest of the sequence."
  - q: "How does 'minimum number of intervals to remove so none overlap' (LeetCode 435) reduce to the same greedy algorithm as 'maximum number of non-overlapping intervals to keep'?"
    a: "They're complements of the same quantity: if the greedy end-time-sort algorithm finds that k intervals can be kept without any overlap, then the remaining (total - k) intervals are exactly the ones that must be removed to eliminate all overlaps, so running the same greedy keep-counting algorithm and subtracting from the total answers the removal-count question directly, with no separate algorithm needed."
deeper:
  - label: "Activity Selection Problem | Greedy Algorithm — GeeksforGeeks"
    url: "https://www.geeksforgeeks.org/dsa/activity-selection-problem-greedy-algo-1/"
related_formulas: []
---

## A different question than merge-intervals

The merge-intervals pattern answers "what's the union of these overlapping ranges."
Greedy interval scheduling (the classic "activity selection problem") answers a
different question: given a set of intervals that may overlap, what's the **maximum
number of them you can keep** such that none of the kept intervals overlap each other?
You're not merging anything — you're choosing a maximal subset and discarding the rest.
The setup is familiar (sort the intervals first), but the sort key and the decision
rule are different, which is exactly why this lesson builds on merge-intervals-pattern
rather than repeating it.

## The greedy rule: sort by end time

Sort all intervals by their **end** time, ascending. Then walk through them once,
keeping a `lastEnd` value initialized to negative infinity. For each interval in
sorted order: if its start is `>= lastEnd`, keep it and update `lastEnd` to its end;
otherwise it overlaps the last interval you kept, so skip it.

**Worked example.** Intervals: `[1,4], [3,5], [0,6], [5,7], [3,9], [6,10], [8,11],
[8,12], [2,14], [12,16]` (a classic textbook set). This set happens to already be
listed in end-time order — the ends read `4, 5, 6, 7, 9, 10, 11, 12, 14, 16`, strictly
increasing — so no reordering is needed before the sweep.

Sweep with `lastEnd = -infinity`:
- `[1,4]`: start 1 `>= lastEnd` → keep. `lastEnd = 4`.
- `[3,5]`: start 3 `< 4` → overlaps, skip.
- `[0,6]`: start 0 `< 4` → skip.
- `[5,7]`: start 5 `>= 4` → keep. `lastEnd = 7`.
- `[3,9]`: start 3 `< 7` → skip.
- `[6,10]`: start 6 `< 7` → skip.
- `[8,11]`: start 8 `>= 7` → keep. `lastEnd = 11`.
- `[8,12]`: start 8 `< 11` → skip.
- `[2,14]`: start 2 `< 11` → skip.
- `[12,16]`: start 12 `>= 11` → keep. `lastEnd = 16`.

Result: 4 intervals kept — `[1,4], [5,7], [8,11], [12,16]` — and this is provably the
maximum possible for this set.

## Why start-time-first fails

Try sorting the same set by start time instead and greedily keeping whatever doesn't
overlap the last kept interval: you'd consider `[0,6]` early (smallest start) and keep
it, locking out `[1,4]`, `[3,5]`, and `[5,7]` all at once because they all overlap
`[0,6]`'s wide span. One long early-starting interval can block multiple short
non-overlapping ones that together would have beaten it. Start time tells you nothing
about how much future room an interval consumes; end time does.

## The exchange argument, informally

Why is end-time-first provably optimal? Take any optimal solution and look at its first
interval. The interval with the earliest end time overall ends at or before that first
interval does — call it `e`. Every interval compatible with the optimal solution's
first choice (i.e., starting after it ends) is also compatible with `e`, since `e` ends
even earlier or at the same time. So swapping `e` in as the first choice can't make the
rest of the optimal solution any harder to complete — the solution stays just as large.
Repeating this swap argument down the line shows that always greedily taking the
earliest-ending compatible interval never does worse than any other strategy, which is
the formal justification for "sort by end time, greedily take what's compatible."

## The LeetCode framing: minimum removals

LeetCode 435, "Non-overlapping Intervals," asks for the minimum number of intervals to
*remove* so none overlap, rather than the max to keep. These are complements: if the
greedy algorithm above finds you can keep `k` intervals, the answer to "how many must
be removed" is simply `total - k`. Same sort, same sweep, same greedy rule — just
report the complement of the count instead of the count itself.
