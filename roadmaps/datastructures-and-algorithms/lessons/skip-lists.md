---
title: "Skip Lists"
description: A probabilistic layered linked list that gets expected O(log n) search, insert, and delete without any tree-rebalancing logic.
roadmap: datastructures-and-algorithms
node: skip-lists
group: advanced-linear-structures
video:
  provider: youtube
  id: "YZ5ef_HiHJ8"
  start: 0
  end: null
creator:
  name: Techdose
  channel_url: https://www.youtube.com/@techdose4u
duration_seconds: 840
level: intermediate
prerequisites: []
takeaways:
  - "A skip list is a sorted linked list with extra 'express lane' levels stacked on top, where each higher level skips over more nodes, letting search jump ahead instead of walking one node at a time."
  - "A node's height (how many levels its 'tower' spans) is chosen randomly at insertion time, typically by flipping a coin and climbing one more level per success — this randomness is what keeps the structure balanced on average without any rebalancing step."
  - "Search, insert, and delete are all expected O(log n), not guaranteed O(log n) — an unlucky run of coin flips can produce a lopsided skip list, but the probability of a bad-enough run to break the log n bound shrinks exponentially."
  - "Unlike a red-black tree or AVL tree, a skip list needs no rotations and no rebalancing invariant to maintain — insert and delete just splice a node into however many levels its random height gave it, which is why skip lists are popular in concurrent and lock-free settings."
  - "Redis uses a skip list as the backing structure for its sorted set (ZSET) type, and LevelDB/RocksDB-style engines use the same layered-list idea (memtables) for their in-memory sorted buffers, specifically because skip lists are simpler to implement correctly under concurrent access than a balanced tree."
check_questions:
  - q: "Why does a skip list give expected O(log n) rather than guaranteed O(log n) search time, and why is that an acceptable tradeoff?"
    a: "Each node's level is set by independent random coin flips at insertion, so there's no invariant enforcing balance the way an AVL tree enforces height limits — a run of bad luck could in principle build every node at height 1, degenerating to a plain linked list with O(n) search. But the probability of enough consecutive high-level nodes failing to appear to break the O(log n) bound shrinks exponentially with n, so in practice, and in expectation over many operations, performance tracks O(log n) extremely closely. The tradeoff buys a much simpler implementation than a tree that guarantees the bound via rotations."
  - q: "What determines how many levels a given node ends up on, and why does that process naturally create fewer nodes at each higher level?"
    a: "When a node is inserted, it starts at level 1 (the base list, which holds every node), then a weighted coin is flipped repeatedly — typically with probability 1/2 (or 1/4 in some implementations) — and the node's tower grows one level taller for each success, stopping on the first failure. Because each additional level requires another coin flip to succeed, roughly half (or a quarter) as many nodes reach each successive level, which is exactly the geometric thinning that makes the top levels short express lanes over the bottom level's full list."
  - q: "Why does a skip list avoid the rebalancing logic that a red-black tree needs, and what does that buy in a concurrent system?"
    a: "A tree's balance is a structural invariant tied to the shape of the whole tree, so inserting or deleting a node can force rotations that touch and restructure other, unrelated nodes to restore that invariant. A skip list has no such invariant — a node's height is decided once, randomly, at insertion, and insert/delete only ever touch the predecessor pointers at the levels that specific node occupies. That locality means updates don't ripple outward to touch unrelated parts of the structure, which makes it far easier to design correct fine-grained locking or lock-free versions, which is exactly why Redis and memtable-style engines favor skip lists for concurrently accessed sorted structures."
deeper:
  - label: "Skip List | Set 1 (Introduction) (GeeksforGeeks)"
    url: "https://www.youtube.com/watch?v=ypod5jeYzAU"
related_formulas: []
---

## A linked list with express lanes

A plain sorted singly linked list gives you O(n) search — you have no choice but to walk node by node until you find or pass your target. A balanced BST fixes this with O(log n) search, but at the cost of rotations and rebalancing invariants that have to survive every insert and delete. A **skip list** gets the same expected O(log n) bound a different way: by stacking extra, sparser linked lists on top of the original one, so a search can skip over large chunks of the list instead of visiting every node.

Picture the base level (level 0) as the full sorted linked list — every element appears here, once. Level 1 contains only some of those elements, linked directly to each other, skipping over the rest. Level 2 contains an even sparser subset of level 1's elements, and so on. Each node that appears on a higher level also has a pointer to its own position at the level directly below it, forming a vertical "tower" that spans however many levels that node reaches.

## Searching: start high, drop down

A search for a target value starts at the topmost level, at a special header node that has no value and exists purely as an entry point. From there, the algorithm moves right along the current level as long as the next node's value is less than the target. When moving right would overshoot (the next node's value is too large, or there is no next node), the search drops down one level and repeats the same rightward scan from its current position. This continues until it drops all the way to level 0, where either the exact node is found, or its absence is confirmed.

Each level skipped over cuts out a large fraction of the nodes that a flat linked list would have had to visit one at a time. With well-balanced level sizes, this works out to a search cost of O(log n), the same bound a balanced tree gives — but without the tree's rebalancing mechanics.

## Building the towers randomly

The structure's balance comes from randomness, not enforcement. When a new node is inserted, its height is decided with a simple randomized procedure: it starts at level 0 (every node is there), and then a biased coin is flipped (commonly with success probability p = 1/2, sometimes 1/4) — on each success the node's tower grows one more level, and the process stops on the first failure. A node that keeps succeeding climbs several levels; most nodes stop after one or two flips.

This naturally produces a geometric thinning across levels: since climbing to level k+1 requires succeeding at roughly a p-probability coin flip from level k, only about a p-fraction of the nodes at level k also appear at level k+1. With p = 1/2, level 0 might have n nodes, level 1 roughly n/2, level 2 roughly n/4, and so on — exactly the halving structure that gives O(log n) levels and O(log n) expected search cost, without ever computing or enforcing balance directly.

Insertion itself works by running the same search described above while recording, at each level, the last node visited before dropping down — these become the "predecessor" pointers that the new node will be spliced in after, at every level up to its randomly chosen height. Deletion runs the same search to locate the node, then unlinks it from every level its tower occupied, using those same predecessor references. Neither operation touches any node outside the path the search walked — there's no analog to a tree rotation that reshapes distant parts of the structure.

## Why it's expected, not guaranteed, and why that's fine in practice

Because each node's height is an independent random outcome, there's no hard guarantee against an unlucky sequence of coin flips producing a lopsided skip list — in the worst theoretical case, every node could land at height 1, degenerating to an ordinary O(n) linked list. But the probability of enough bad luck to meaningfully break the O(log n) bound shrinks exponentially as n grows, so in both expectation and overwhelming practical likelihood, a skip list behaves like a balanced structure.

## Why Redis and storage engines reach for it

Redis implements its sorted set (`ZSET`) type as a skip list paired with a hash table, specifically because a skip list's inserts and deletes touch only the predecessor chain for one node's tower — nothing analogous to a tree rotation ripples outward to restructure unrelated parts of the data. That locality makes correct concurrent and lock-free implementations substantially easier to get right than an equivalent balanced tree, which is also why LSM-tree storage engines in the LevelDB/RocksDB family use the same layered-list idea for their in-memory memtable: a sorted, concurrently-writable buffer that needs to support fast inserts and ordered scans without tree-rebalancing overhead.
