---
title: "Segment Trees"
description: A binary tree over an array that answers any range sum/min/max query in O(log n) and, with lazy propagation, applies range updates just as fast.
roadmap: datastructures-and-algorithms
node: segment-trees
group: priority-structures-and-range-queries
video:
  provider: youtube
  id: "ZBHKZF5w4YU"
  start: 0
  end: null
creator:
  name: Tushar Roy - Coding Made Simple
  channel_url: https://www.youtube.com/@tusharroy2525
duration_seconds: 1664
level: intermediate
prerequisites:
  - skip-lists
  - trees-bsts-and-heaps
takeaways:
  - "A segment tree is a binary tree built over an array where every node stores an aggregate (sum, min, or max) of a contiguous range, the root covers the whole array, and each leaf covers a single element."
  - "Build is a recursive divide-and-conquer: a node's range splits in half, each half is built recursively, and the node's value is combined from its two children — this takes O(n) total, not O(n log n), because the total work across all levels is one pass over the array."
  - "A point update only touches the O(log n) nodes on the root-to-leaf path for that index, recomputing each ancestor's aggregate on the way back up."
  - "A range query walks down from the root, and at each node either returns immediately (the node's range sits fully inside the query), recurses into both children (the ranges partially overlap), or returns an identity value (no overlap) — visiting O(log n) nodes total because any query range decomposes into at most O(log n) 'canonical' node ranges."
  - "Lazy propagation defers range updates: a node marks itself with a pending update and stops, instead of recursing all the way to the leaves, and only pushes that pending update down to its children the next time a query or update actually needs to descend past it."
  - "Without laziness, a range update that touches every element costs O(n); with lazy propagation it costs O(log n), because the update is applied to O(log n) whole-range nodes and left undistributed until needed."
check_questions:
  - q: "Why does a range query only ever visit O(log n) nodes, rather than one node per element in the range?"
    a: "Because any contiguous range over n elements can be exactly covered by at most about 2*log2(n) 'canonical' nodes — nodes whose own range lies entirely inside the query range — and the query algorithm stops descending the instant it hits one of those nodes. It only keeps recursing into both children when a node's range straddles the query boundary, and there are only O(log n) such boundary-straddling nodes per side."
  - q: "What problem does lazy propagation solve, and what would break if you skipped the 'push down pending updates before recursing further' step?"
    a: "It solves the cost of range updates: without it, applying +5 to a range of size n means touching all n affected leaves, which is O(n) per update. Lazy propagation marks a fully-covered node and stops, paying only O(log n). But if you then query or update a *sub*-range of that node without first pushing the pending mark down to its children, the children's stored aggregates would be stale — the query would read values that don't reflect the pending update, producing a wrong answer."
  - q: "Why is a sum segment tree's 'identity value' for no-overlap nodes different from a min segment tree's identity value, and why does that distinction matter?"
    a: "A sum tree returns 0 for no-overlap (adding 0 doesn't change a running total), while a min tree returns +infinity (a value that can never win a minimum comparison) — each identity element must be neutral for that specific combining operation. Using the wrong identity, like returning 0 from a min query's no-overlap branch, would corrupt every query, since 0 would incorrectly win against any positive minimum."
deeper:
  - label: "Lazy Propagation Segment Tree (Tushar Roy, range updates in depth)"
    url: "https://www.youtube.com/watch?v=xuoQdt5pHj0"
related_formulas: []
---

## The shape of the tree

Given an array of n elements, a segment tree is a complete binary tree where the root represents the range [0, n−1], and recursively, a node covering [lo, hi] splits into a left child covering [lo, mid] and a right child covering [mid+1, hi], down to leaves that each cover a single index. Stored as an array (like a binary heap), node i's children sit at 2i+1 and 2i+2, and the tree needs about 4n slots to stay safely within bounds for any n.

Each internal node caches an aggregate of its range — for a sum tree, the sum of its children's sums; for a min tree, the minimum of its children's minimums. Building the tree is one recursive pass: recurse left, recurse right, then combine — O(n) total work, since each of the roughly 2n nodes does O(1) work once its children are done.

## Query: decomposing a range into canonical pieces

To answer "what's the sum of [L, R]?", start at the root and compare the node's range to [L, R]:

- **No overlap** (node's range is entirely outside [L, R]): return the identity (0 for sum, +∞ for min, −∞ for max) — contributes nothing.
- **Total overlap** (node's range sits entirely inside [L, R]): return the node's cached aggregate immediately — no need to look deeper.
- **Partial overlap**: recurse into both children and combine their results.

The key fact that makes this O(log n): a query range only ever causes "partial overlap" recursion along two root-to-leaf paths (the ones ending near L and near R); everywhere else, the recursion bottoms out at a total-overlap or no-overlap node after a bounded number of steps. The query touches O(log n) total-overlap nodes, each contributing in O(1), plus O(log n) nodes along the two boundary paths.

## Worked example

Array: [2, 5, 1, 4, 9, 3]. A sum segment tree's root covers [0,5] = 24; its children cover [0,2] = 8 and [3,5] = 16; and so on down to six leaves. Querying sum(1, 4) — expected 5+1+4+9 = 19 — starts at the root [0,5], which partially overlaps [1,4], so it recurses. [0,2] partially overlaps [1,4] (recurse again into [0,1] and [2,2]); [3,5] partially overlaps [1,4] too (recurse into [3,4] and [5,5]). [2,2] is totally inside [1,4] → return 1. [3,4] is totally inside → return 4+9=13. [0,1]'s right half [1,1]=5 is totally inside → return 5; its left half [0,0] holds value 2 but is no-overlap → return 0. Combined, [0,1] contributes 5. [5,5] is no-overlap → return 0. Summing the contributing pieces: 5 + 1 + 13 + 0 = 19. Only a handful of the tree's nodes were ever touched, not all six leaves.

## Updates, and lazy propagation for ranges

A point update (set index i to a new value) walks straight down to leaf i, changes it, then walks back up recomputing each ancestor's aggregate from its two children — O(log n).

A *range* update (add 5 to every element in [L, R]) is where naive segment trees get expensive: applied recursively all the way to leaves, it's O(n) in the worst case, because up to n leaves individually change. Lazy propagation avoids this by letting a node that is totally covered by the update range absorb the update into a "pending" tag and stop recursing — its cached aggregate is updated as if the push had happened (e.g., add 5 × range-size to a sum node), but its children are left untouched and unaware. Only when a later operation needs to descend *past* that node — because some other query or update only partially overlaps its range — does the node "push down" its pending tag to its two children (updating their aggregates and giving them their own pending tags) before continuing. This way, both range updates and range queries cost O(log n): the tree only ever does real work proportional to the depth, and defers the rest until it's actually needed.
