---
title: "Trees, BSTs, and Heaps"
description: Binary trees, the binary search tree property, the three traversal orders, and how a heap keeps the smallest or largest element one step away.
roadmap: datastructures-and-algorithms
node: trees-bsts-and-heaps
group: core-data-structures
video:
  provider: youtube
  id: "oSWTXtMglKE"
  start: 0
  end: null
creator:
  name: HackerRank
  channel_url: https://www.youtube.com/@HackerrankOfficial
duration_seconds: 480
level: beginner
prerequisites:
  - hash-tables-and-tries
takeaways:
  - "A binary tree is a hierarchy where every node has at most two children; a binary search tree adds the rule that everything in a node's left subtree is smaller and everything in its right subtree is larger."
  - "That BST ordering rule is what makes search, insert, and delete O(log n) on average — each comparison eliminates roughly half the remaining tree, the same halving idea behind binary search."
  - "A BST's O(log n) average case is not a guarantee: inserting already-sorted data into a naive BST produces a tree shaped like a straight line, with every operation degrading to O(n)."
  - "In-order traversal of a BST visits every node in sorted order, which is a direct consequence of the left-smaller, right-larger rule and a useful way to confirm a tree is a valid BST."
  - "A heap is a different kind of ordered tree — it only guarantees a parent is smaller (min-heap) or larger (max-heap) than its children, which is weaker than BST ordering but is exactly what a priority queue needs: O(1) access to the smallest or largest element and O(log n) insert/remove."
check_questions:
  - q: In a binary search tree, why does deciding "go left or go right" at each node let you skip checking half the remaining nodes?
    a: Every node in a BST's left subtree is guaranteed to be smaller than the node, and everything in its right subtree is guaranteed to be larger. So comparing the target value against the current node tells you which entire subtree it could possibly be in — the other subtree, whatever its size, can be ruled out in one comparison, the same halving logic as binary search over a sorted array.
  - q: Pre-order, in-order, and post-order traversal all visit the same three parts of a tree — a node, its left subtree, and its right subtree — but in different sequences. Which order visits a BST's values from smallest to largest, and why?
    a: In-order traversal (left subtree, then the node, then right subtree) visits a BST in sorted order, because at every node it fully exhausts everything smaller (the left subtree) before visiting the node itself, then everything larger (the right subtree) — which, applied recursively, produces a strictly increasing sequence.
  - q: A min-heap only guarantees that a parent is smaller than its children — not that the whole tree is ordered left-to-right like a BST. Why is that weaker guarantee actually the right tradeoff for a priority queue?
    a: A priority queue only ever needs to know the single smallest (or largest) element quickly, not find an arbitrary value by search — and the heap property puts that element at the root in O(1) to read. Because the heap doesn't also need to maintain BST-style left/right ordering throughout the tree, it can stay a complete, array-packed, always-balanced shape, which keeps insert and remove-the-minimum both at O(log n) without any rebalancing logic.
related_formulas: []
---


## Trees as a generalization of a linked list

A linked list chains nodes one after another; a **tree** lets a node branch into more than one "next." A **binary tree** caps that branching at two: every node has at most a left child and a right child. That simple shape underlies a huge amount of computer science, but it's only useful for fast search once an ordering rule is added on top.

## The binary search tree property

A **binary search tree (BST)** adds exactly one rule: for every node, everything in its left subtree is smaller than the node, and everything in its right subtree is larger. That's the entire definition — but it has a powerful consequence. To search for a value, start at the root and compare: if the target is smaller, the answer (if it exists) must be somewhere in the left subtree, so the entire right subtree can be ignored; if it's larger, the mirror image holds. Each comparison eliminates roughly half of what's left, the same divide-and-conquer idea as binary search over a sorted array — except here the "array" is a tree shape that also supports fast insertion and deletion, which a sorted array does not (inserting into a sorted array means shifting everything after the insertion point, an O(n) cost a BST avoids).

Inserting into a BST follows the same left/right comparisons used for search, continuing until an empty spot is found and the new node is attached there. Deletion is the fiddliest of the three operations: deleting a leaf is trivial (just remove it), deleting a node with one child means splicing that child up into the deleted node's place, and deleting a node with two children means finding its in-order successor (the smallest value in its right subtree), copying that value into the node being deleted, and then deleting the successor instead — which is guaranteed to be a simpler case, since the smallest value in a subtree can have at most one child.

## The catch: shape depends on insertion order

All of this "eliminate half each time" reasoning assumes the tree is roughly balanced — its height close to log₂(n). But nothing in the plain BST definition enforces that. Insert values in already-sorted order — 1, 2, 3, 4, 5 — and every new value is larger than everything already in the tree, so each one becomes the right child of the previous one. The result is a tree that is really a linked list wearing a tree's clothing: height n instead of log n, and every operation degrading to O(n). This is exactly the problem the next lesson's balanced search trees (AVL trees, red-black trees) exist to solve, by actively restructuring the tree during insertion and deletion to keep its height close to log n regardless of insertion order.

## Walking a tree: three orders

A traversal visits every node in a tree exactly once, and the three classic orders differ only in when they visit the current node relative to its subtrees:

- **Pre-order** (node, then left, then right) visits a node before either of its subtrees — useful for copying a tree's structure, since parents are processed before children.
- **In-order** (left, then node, then right) visits a node between its subtrees. Applied to a BST specifically, this produces every value in sorted order — a direct consequence of "left is smaller, right is larger" applied recursively at every level.
- **Post-order** (left, then right, then node) visits a node after both subtrees — useful whenever children must be fully processed before their parent, such as safely deleting an entire tree (delete both children before the node that pointed to them) or evaluating an expression tree (evaluate both operands before applying the operator).

## Heaps: a looser rule, built for a different job

A **heap** is also a binary tree, but it enforces a different, weaker rule than a BST: in a **min-heap**, every parent is smaller than or equal to both of its children (a **max-heap** reverses the inequality). Note what this does *not* say — there's no relationship enforced between a node's left and right children, or between nodes in different subtrees. That weaker guarantee is deliberate: a heap is built to answer one question extremely fast — "what's the smallest (or largest) element overall?" — which is always sitting at the root, an O(1) read. A BST could also find the minimum, but it would need to walk all the way down the left spine of the tree to get there.

In exchange for that weaker ordering, a heap can keep a much stricter *shape*: it's a **complete binary tree**, meaning every level is fully filled except possibly the last, which fills left to right with no gaps. That completeness means a heap can be stored directly in a plain array with no pointers at all — a node at array index i has its children at indices 2i+1 and 2i+2 — and it guarantees the tree's height stays at log₂(n) automatically, with no rebalancing logic required. Inserting a new element adds it at the next open array slot and then "bubbles it up" by swapping with its parent while it violates the heap property; removing the root swaps in the last element and "sifts it down" the same way. Both operations are O(log n), and that combination — O(1) peek at the extreme value, O(log n) insert and remove — is precisely what a **priority queue** needs, which is why heaps are the standard way to implement one. Priority queues built on heaps reappear later in this roadmap as the core data structure behind Dijkstra's shortest-path algorithm and Prim's minimum-spanning-tree algorithm.
