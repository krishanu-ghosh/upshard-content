---
title: "Balanced Search Trees"
description: Why an unbalanced BST can degrade to a linked list, and how AVL trees, red-black trees, and B-trees stay fast through rotations.
roadmap: datastructures-and-algorithms
node: balanced-search-trees
group: core-data-structures
video:
  provider: youtube
  id: "q4fnJZr8ztY"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 780
level: intermediate
prerequisites:
  - trees-bsts-and-heaps
takeaways:
  - A plain binary search tree's O(log n) guarantee only holds if the tree stays roughly balanced; adversarial or just unlucky insertion order can make it degenerate into an O(n) linked list.
  - A rotation is a local restructuring — reattaching a small number of pointers around two nodes — that changes the tree's shape without breaking the BST ordering property, which is what lets a tree rebalance itself without a full rebuild.
  - An AVL tree enforces a strict balance rule (the heights of any node's two subtrees differ by at most 1) and rebalances with rotations after every insert or delete, giving the tightest height bound of the common self-balancing trees.
  - A red-black tree enforces a looser balance rule via a color invariant on nodes, which means fewer rotations on average than an AVL tree — a common tradeoff in language standard libraries, which favor red-black trees for faster inserts at the cost of a slightly taller tree.
  - A B-tree generalizes the idea to nodes that hold many keys and many children rather than just two, which minimizes the number of disk reads needed for a lookup — the reason B-trees (not binary trees) are the standard structure behind database and filesystem indexes.
check_questions:
  - q: If you insert the values 1, 2, 3, 4, 5 into a plain binary search tree in that order, what shape does the tree end up with, and why is that bad?
    a: Each new value is larger than everything already in the tree, so it always becomes the right child of the previously inserted node — the tree degenerates into a straight line with height 5, functionally a linked list. Every search, insert, and delete on it costs O(n) instead of the O(log n) a balanced tree would give, because there's no branching left to eliminate half the remaining nodes at each step.
  - q: What is a rotation, and why does performing one not break the binary-search-tree ordering property?
    a: A rotation takes a node and one of its children and swaps their positions, reattaching a small number of pointers (and reassigning a subtree in between) so the shorter side gets taller and the taller side gets shorter. It preserves the BST property because it only ever moves a subtree to a position where the ordering still holds — the in-order sequence of the tree's values is exactly the same before and after.
  - q: Why do databases index their tables with B-trees rather than binary search trees, even though both give logarithmic search time?
    a: A B-tree node holds many keys and many children, not just two, so a B-tree over the same data is far shallower than a binary tree would be — often just three or four levels for millions of rows. Since each level typically corresponds to one disk read, and disk reads are by far the slowest part of a database query, a shallower tree directly means fewer disk reads per lookup, which is the dominant cost a database is optimizing for.
related_formulas: []
---

## Why "balanced" is the whole game

The previous lesson's binary search tree gives O(log n) search, insert, and delete — but only on the assumption that the tree's height stays close to log₂(n). Nothing in the plain BST definition guarantees that. Insert data that happens to be already sorted, or that an adversary deliberately chose to be worst-case, and a naive BST degenerates into a tree that is really a straight line: every node has only a right child (or only a left child), height n instead of log n, and every operation back to O(n) — no better than scanning a linked list. A **self-balancing search tree** is a BST with extra bookkeeping that actively restructures itself after every insert and delete, so its height never strays far from log₂(n) no matter what order the data arrives in.

## The move that makes rebalancing possible: rotation

The tool every self-balancing tree relies on is the **rotation** — a local, constant-time restructuring of a small number of nodes that changes the tree's shape without disturbing its ordering. In a right rotation around a node X with left child Y, Y takes X's place, X becomes Y's right child, and the subtree that used to be Y's right child becomes X's new left child. It sounds fiddly described in words, but the key property is simple: the in-order traversal of the tree — the sorted sequence of its values — is identical before and after the rotation. Nothing about *what's stored* changes; only *how it's shaped* changes, specifically shifting height away from one side and onto the other. A left rotation is the mirror image. Chaining a few of these together after an insertion or deletion is enough to restore balance anywhere a tree has gotten lopsided.

## AVL trees: strict balance, more rotations

An **AVL tree** (named for its inventors, Adelson-Velsky and Landis) enforces the strictest common balance rule: for every node, the heights of its left and right subtrees differ by at most 1. After every insertion or deletion, the tree walks back up from the changed node toward the root, checking this balance factor at each ancestor. The moment it finds a node where the heights differ by more than 1, it applies one or two rotations (there are four cases — left-left, right-right, left-right, right-left — but all reduce to combinations of the single rotation described above) to restore the invariant, then continues checking further up. Because the balance rule is strict, an AVL tree's height is always very close to log₂(n), which makes it the fastest of the common self-balancing trees for *lookups*. The cost is that strict rule has to be re-verified and potentially fixed at every level on the way back up after every single insert or delete, which is more rebalancing work than a looser scheme needs.

## Red-black trees: a looser rule, fewer rotations

A **red-black tree** achieves balance a different way: every node is colored red or black, and a small set of color-based rules (the root is black, a red node can't have a red child, every path from a node to its descendant null leaves passes through the same number of black nodes) together guarantee that the longest root-to-leaf path is never more than twice the shortest one. That's a looser bound than AVL's — a red-black tree can end up somewhat taller — but enforcing it typically needs fewer rotations per insert or delete, because many violations can be fixed just by recoloring nodes rather than restructuring the tree. That tradeoff — slightly taller tree, cheaper to keep balanced — is why red-black trees are the structure behind many language standard libraries that need an ordered map or set (C++'s `std::map`, Java's `TreeMap`), where insert/delete frequency often matters more than squeezing out the last bit of lookup speed.

## 2-3 trees, 2-3-4 trees, and B-trees: more than two children

A different generalization relaxes the "binary" part instead of the balance rule. A **2-3 tree** lets a node hold either one key with two children or two keys with three children; a **2-3-4 tree** extends that to up to three keys and four children. Allowing more keys per node means fewer levels are needed to store the same amount of data, since each level branches more ways. A **B-tree** takes this all the way: a node can hold many keys and many children — often dozens or hundreds, tuned to match the size of a disk page — which keeps the tree extremely shallow even over millions of entries. That shallowness is exactly what databases and filesystems need: each level of the tree typically costs one disk read, and disk reads are orders of magnitude slower than comparisons in memory, so minimizing levels (not minimizing comparisons) is the thing worth optimizing for. This is the direct reason B-trees, not binary search trees, are the standard index structure inside relational databases — a connection this roadmap returns to in the lesson on database indexes.

## Other structures in the same family

Two more structures solve similar balance problems in different shapes, worth knowing by name even without full detail here: a **K-D tree** is a BST generalized to multiple dimensions, splitting on a different coordinate axis at each level, used for nearest-neighbor search in spatial data; a **skip list** achieves balanced-tree-like O(log n) search and insert using multiple layered linked lists with randomized "express lanes" instead of rotations at all, trading a guaranteed worst case for a simpler, lock-friendlier implementation that some databases use for concurrent access.
