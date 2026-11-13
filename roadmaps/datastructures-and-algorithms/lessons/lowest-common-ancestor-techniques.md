---
title: "Lowest Common Ancestor Techniques"
description: Finding the deepest shared ancestor of two tree nodes fast, via binary lifting's O(n log n) preprocessing and O(log n) queries.
roadmap: datastructures-and-algorithms
node: lowest-common-ancestor-techniques
group: more-graph-algorithms
video:
  provider: youtube
  id: "02zM-QoKoPg"
  start: 0
  end: null
creator:
  name: Fluent Algorithms
  channel_url: https://www.youtube.com/@fluentalgorithms4847
duration_seconds: 911
level: intermediate
prerequisites: ["segment-trees", "trees-bsts-and-heaps"]
takeaways:
  - "The lowest common ancestor (LCA) of two nodes u and v in a rooted tree is the deepest node that is an ancestor of both — finding it naively by walking both nodes up to the root takes O(depth) per query, which is slow when the tree is a long chain and there are many queries."
  - "Binary lifting precomputes, for every node, its 2^k-th ancestor for every power of two k up to log2(n), stored in a table up[node][k], built with the recurrence up[node][k] = up[up[node][k-1]][k-1] — the 2^k-th ancestor is the 2^(k-1)-th ancestor of the 2^(k-1)-th ancestor."
  - "Building the full up[][] table takes O(n log n) time and space, since there are O(log n) powers of two for each of the n nodes, but afterward any node's 2^k-th ancestor is a single array lookup instead of a walk."
  - "An LCA query first lifts the deeper node up to the same depth as the shallower one (using the binary representation of the depth difference to jump in O(log n) steps), then lifts both nodes together in decreasing powers of two, stopping just before they'd meet — the final step lands one node below the answer, so the LCA is one more hop up from either."
  - "A second, equally standard technique reduces LCA to a Range Minimum Query: an Euler tour of the tree records every node each time DFS enters or backtracks through it, and the LCA of u and v is the minimum-depth node between their first occurrences in that tour, answerable with a sparse table in O(1) per query after O(n log n) preprocessing."
check_questions:
  - q: "Why does binary lifting build its ancestor table using powers of two instead of, say, precomputing every node's ancestor at every possible distance?"
    a: "Precomputing every distance would take O(n^2) space and time in the worst case (a chain of n nodes has distances up to n). Powers of two give a far more compact basis: any distance can be written as a sum of at most log2(n) powers of two (its binary representation), so storing just the O(log n) power-of-two ancestors per node lets any jump distance be assembled by combining a handful of lookups, trading an exponential blow-up for a logarithmic one."
  - q: "In an LCA query, why must the algorithm first equalize the depths of u and v before jumping them together in powers of two?"
    a: "The 'jump both up by decreasing powers of two, stop just before they meet' step only works correctly when the two nodes are the same number of edges from their actual LCA — if one starts deeper than the other, jumping them by the same amount each step won't land them in sync, and the algorithm could overshoot past the true LCA or compare nodes that aren't on a common footing at all. Equalizing depth first guarantees both nodes are racing toward the LCA from the same altitude."
  - q: "During the simultaneous lifting phase, the algorithm deliberately stops at the step just before u and v would become equal, then takes one final step up. Why not just jump until they become equal and stop there?"
    a: "If u and v are lifted to the point where they become equal mid-algorithm, that equal node might be far above the actual LCA, not the LCA itself — the algorithm can't tell, from a single comparison, whether it has reached the LCA or overshot it. By always jumping only when the result would keep u and v distinct, the algorithm guarantees it stops with both nodes as direct children of the LCA (or just below it), so one more step up from either is provably the exact answer."
deeper:
  - label: "Lowest Common Ancestor (LCA) Problem | Eulerian path method — WilliamFiset"
    url: "https://www.youtube.com/watch?v=sD1IoalFomA"
related_formulas: []
---

## The problem

Given a rooted tree and two nodes `u` and `v`, their **lowest common ancestor (LCA)** is the deepest node that lies on the path from the root to both of them. It's the point where the paths from `u` and `v` back to the root first merge. LCA queries show up constantly in practice — computing the distance between two nodes in a tree (which reduces to `depth[u] + depth[v] - 2*depth[LCA(u,v)]`), answering "is `u` an ancestor of `v`," and version-control-style common-ancestor lookups in commit graphs are all built on fast LCA.

The naive approach — walk both nodes up toward the root one parent-pointer at a time, keeping track of visited nodes, until you find a shared one — costs O(depth) per query. On a tree that's a long chain, depth can be O(n), so answering many queries this way is far too slow. The goal is to answer each query in O(log n) time after some one-time preprocessing.

## Binary lifting

The core trick: precompute, for every node, pointers to its ancestor at every power-of-two distance, not just its immediate parent. Build a table `up[node][k]`, where `up[node][k]` is the ancestor of `node` that is `2^k` steps up. The base case is `up[node][0]` = the direct parent. Every other entry follows the recurrence:

```
up[node][k] = up[ up[node][k-1] ][k-1]
```

In words: the ancestor `2^k` steps up is the ancestor `2^(k-1)` steps up *from* the ancestor that's already `2^(k-1)` steps up. This lets the table be filled level by level, and since `k` only needs to range up to `log2(n)`, the whole table has `n * log(n)` entries, each computed in O(1) from a smaller entry — O(n log n) total preprocessing time and space.

Once built, finding the ancestor of any node at *any* distance `d` is fast: write `d` in binary, and for each set bit at position `k`, jump using `up[node][k]`. Since `d` has at most `log2(n)` bits, this takes O(log n) jumps instead of `d` individual parent-steps.

## Answering an LCA query

Given `u` and `v`, with depths `depth[u]` and `depth[v]`:

1. **Equalize depth.** Suppose `depth[u] > depth[v]`. Lift `u` up by exactly `depth[u] - depth[v]` steps using the binary-jump trick above, so both nodes are now at the same depth. If they're now the same node, that node is the LCA — one of the original nodes was already an ancestor of the other.

2. **Lift together, in decreasing powers of two.** Starting from the largest power of two down to `2^0`, for each power `2^k`, check whether `up[u][k] != up[v][k]` (i.e., jumping both up by `2^k` would still leave them distinct). If so, jump both: `u = up[u][k]`, `v = up[v][k]`. If jumping would make them equal or cross past each other, skip that power and try smaller ones.

3. **Finish.** After trying every power of two this way, `u` and `v` end up as two *different* nodes that share the same parent — which means `up[u][0]` (equivalently `up[v][0]`) is the LCA.

The reason step 2 deliberately avoids jumping when it would make `u` and `v` equal is subtle but important: if they became equal partway through, you couldn't tell whether you'd landed exactly on the LCA or jumped past it into a shared ancestor higher up. By only taking jumps that keep them apart, the algorithm guarantees that when it finishes, `u` and `v` are the two children (possibly several hops removed) that most tightly flank the LCA from below, and a single final parent lookup is provably correct.

Each query does O(log n) work in step 1 and O(log n) in step 2, so binary lifting answers each LCA query in O(log n) time after O(n log n) preprocessing — a dramatic improvement over the naive O(depth) approach for large or skewed trees.

## The alternative: Euler tour + RMQ

A different, equally standard technique reduces LCA to a **Range Minimum Query**. Do a DFS Euler tour of the tree, recording a node's label every time the DFS visits or backtracks through it (a node with `c` children appears `c+1` times in the tour). Alongside this, record each visit's depth.

The key fact: the LCA of `u` and `v` is the node with the *minimum depth* appearing anywhere between `u`'s first occurrence and `v`'s first occurrence in the tour, because the tour must pass back through their common ancestor to get from one subtree to the other. Answering "minimum depth in this tour range" is a classic RMQ problem, solvable with a **sparse table** in O(n log n) preprocessing and O(1) per query — asymptotically even better than binary lifting's O(log n) queries, at the cost of a fiddlier implementation (building the Euler tour and depth arrays correctly, and handling the sparse table's overlapping-range trick).

Both techniques are worth knowing. Binary lifting's table also generalizes naturally to answering other "k-th ancestor" and path-aggregate queries that the Euler tour method doesn't directly support.
