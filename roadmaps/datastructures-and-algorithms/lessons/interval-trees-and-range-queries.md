---
title: "Interval Trees and Range Queries"
description: Augment a BST with one extra field — the max endpoint in its subtree — and it can report every interval overlapping a query in O(log n + k).
roadmap: datastructures-and-algorithms
node: interval-trees-and-range-queries
group: priority-structures-and-range-queries
video:
  provider: youtube
  id: "7oaLSUDxpQs"
  start: 0
  end: null
creator:
  name: TrustFrame
  channel_url: https://www.youtube.com/@TrustFrame09
duration_seconds: 412
level: intermediate
prerequisites:
  - fenwick-trees
takeaways:
  - "An interval tree stores a set of intervals, each inserted into a BST keyed by its low endpoint, so you keep ordinary BST search/insert/delete plus one extra capability: finding every interval that overlaps a query."
  - "Each node is augmented with a 'max' field equal to the largest high-endpoint anywhere in its subtree (itself included), maintained the same way a size or height field would be — recomputed from a node's children whenever the subtree below it changes."
  - "Two intervals [a,b] and [c,d] overlap exactly when a <= d and c <= b — the search algorithm uses this single inequality, pruning entire subtrees instead of checking every stored interval."
  - "To search for intervals overlapping a query range, at each node you check the node's own interval, then only recurse left if the left child's max field is >= the query's low endpoint (otherwise nothing on that side can possibly reach far enough), and recurse right otherwise — this gives O(log n + k) for k matches, versus O(n) for a linear scan."
  - "This is a different tool from a segment tree: a segment tree answers queries about a fixed set of *points* or array positions (sum/min/max over an index range), while an interval tree answers queries about a dynamic set of *intervals themselves* (which stored intervals touch this point or range)."
  - "Real systems use this for calendar conflict detection (does a new meeting overlap any existing one on this calendar?) and genomic range queries (which annotated gene regions overlap this sequenced read?) — both are 'find all stored ranges touching this query' problems at scale."
check_questions:
  - q: "Why does the max-endpoint field let the search skip an entire subtree instead of just one node?"
    a: "The max field is the largest high-endpoint anywhere in that subtree, so if even that maximum is smaller than the query's low endpoint, every single interval in the subtree ends before the query starts — none of them can possibly overlap, no matter how the subtree is shaped. That single comparison substitutes for checking every interval in the subtree individually."
  - q: "Two intervals [1,5] and [5,10] — do they overlap, and how does the a <= d and c <= b test confirm it?"
    a: "Yes, they share the point 5. With [a,b]=[1,5] and [c,d]=[5,10]: a<=d is 1<=10 (true) and c<=b is 5<=5 (true), so both conditions hold and the test reports overlap. This also shows the test treats touching endpoints as overlapping, which is the usual convention for closed intervals — a tree meant to treat adjacent intervals as non-overlapping would need strict inequalities instead."
  - q: "Why is an interval tree the right structure for calendar conflict detection but a segment tree is not a natural fit for the same job?"
    a: "A calendar holds a dynamic, sparse set of meeting intervals with endpoints anywhere on a continuous time axis, and the query is 'which stored meetings overlap this new one' — exactly what the max-augmented BST search answers in O(log n + k). A segment tree needs a fixed, pre-known index range to build its tree over and is built to aggregate values per position, not to store and search a changing collection of arbitrary intervals, so you'd need awkward workarounds (coordinate compression, rebuilding) to force the same job onto it."
deeper:
  - label: "Interval trees: An example of RB Tree Augmentation (longer, full augmentation walkthrough)"
    url: "https://www.youtube.com/watch?v=iaOVnU2NhYg"
related_formulas: []
---

## Augmenting a BST instead of inventing a new structure

An interval tree is not a new tree shape — it's an ordinary balanced BST (AVL, red-black, whatever you already have) where each node stores an interval [low, high] instead of a single key, and the tree is keyed and kept ordered by `low`. On top of that, every node carries one extra piece of derived data: `max`, the largest `high` endpoint found anywhere in that node's subtree, including the node itself.

That single augmented field is maintained exactly like a subtree-size or subtree-height counter in any other augmented BST: `node.max = max(node.high, left.max if exists, right.max if exists)`, recomputed bottom-up whenever an insert, delete, or rotation changes what's below a node. Because rotations only touch O(1) nodes and each recomputation is O(1), augmentation adds no asymptotic cost — insert and delete stay O(log n).

## The overlap test, and why it lets you prune

Two intervals [a,b] and [c,d] overlap if and only if a ≤ d and c ≤ b — equivalently, they *fail* to overlap only if one ends entirely before the other starts. This is the only test the search needs.

To find intervals overlapping a query [qlo, qhi], start at the root and at each node:

1. Check whether the node's own interval overlaps the query; if so, report it.
2. If the left child exists and `left.max ≥ qlo`, recurse left — something in that subtree might still reach far enough to overlap.
3. Recurse right if the node's own `low ≤ qhi`. Anything further right only gets larger `low` values, so if even this node's `low` is already past the query's end, nothing further right can start early enough either.

The pruning power comes entirely from step 2. If `left.max < qlo`, every interval in the left subtree ends before the query even starts, so the whole subtree is skippable in O(1) — no need to visit any of its nodes individually. That's what turns an O(n) linear scan over all stored intervals into an O(log n + k) search, where k is the number of overlaps actually reported (you still spend time listing the results you found).

## Worked example

Intervals keyed by low endpoint: [15,20], [10,30], [17,19], [5,12], [12,15], [30,40]. Build a BST on `low` (simplified, unbalanced for clarity): root [15,20] (max=40, since its subtree ends up containing [30,40]); left child [10,30] (max=30); left-left [5,12] (max=12); left-right [12,15] (max=15); right child [30,40] (max=40); and [17,19] tucked under [15,20] appropriately — the exact shape depends on the balancing scheme, but the max fields always reflect "furthest-reaching interval in this subtree."

Query: find anything overlapping [14,16].

- At root [15,20]: overlap test (15≤16 and 14≤20) → true, report [15,20]. Left child exists ([10,30], max=30 ≥ 14) → recurse left.
- At [10,30]: overlap test (10≤16 and 14≤30) → true, report [10,30]. Its left child [5,12] has max=12, and 12 < 14, so skip that entire subtree without visiting it — nothing ending by 12 can overlap something starting at 14.
- Check right child [12,15] (max=15 ≥ 14) → recurse: overlap test (12≤16, 14≤15) → true, report [12,15].

The search never ran an individual comparison against [17,19] or [30,40] — the max fields ruled them out, despite the query only being 2 units wide.

## Where this shows up

A calendar app checking "does this new meeting conflict with anything already on the calendar?" is exactly an interval-overlap query against a dynamic set of booked ranges. Genomic tools run the same query at much larger scale: given millions of annotated gene or feature regions on a chromosome, "which annotations overlap this newly sequenced read's coordinates?" gets answered the same way. Both problems are "search a changing set of ranges for everything touching a query range" — precisely what the max-augmented BST was built to answer in O(log n + k) instead of a full linear scan.
