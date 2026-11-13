---
title: "Union-Find & Disjoint Sets"
description: Two small optimizations turn a tree-of-sets data structure into one of the fastest tools in algorithms — near-constant-time union and find.
roadmap: datastructures-and-algorithms
node: union-find-and-disjoint-sets
group: union-find-and-graph-structure
video:
  provider: youtube
  id: "PGZ64ob440I"
  start: 0
  end: null
creator:
  name: Stable Sort
  channel_url: https://www.youtube.com/@stablesort
duration_seconds: 352
level: intermediate
prerequisites: [graph-representation]
takeaways:
  - "A disjoint-set-union (DSU) structure represents a partition of elements into groups as a forest, where each tree's root is the group's canonical representative."
  - "Union by rank (or by size) always attaches the shorter/smaller tree under the taller/bigger one, which keeps trees from degenerating into long chains."
  - "Path compression flattens the tree on every find by re-pointing every node along the search path directly to the root, so future finds on those nodes are instant."
  - "Neither optimization alone is enough for near-O(1) performance: union-by-rank bounds tree height logarithmically, but only path compression collapses that height toward 1 over repeated calls."
  - "Combined, the amortized cost per operation is O(alpha(n)), the inverse Ackermann function, which is at most 4 or 5 for any n you could ever construct — effectively constant."
  - "DSU is the engine behind Kruskal's MST (skip an edge if its endpoints are already unioned), cycle detection in undirected graphs, and online connected-components queries where edges only ever get added."
check_questions:
  - q: "Why does union-by-rank alone, without path compression, still leave find() taking O(log n) instead of O(1)?"
    a: "Union-by-rank only guarantees the tree's height never exceeds log n (by always hanging the shorter tree under the taller one), so a find still has to walk up to log n parent pointers to reach the root — it bounds the height, but doesn't shrink it."
  - q: "Why does path compression alone, without union by rank, still risk slow operations?"
    a: "Without a rule for which root wins during union, a long chain can still get built (e.g. always attaching the new root under the old one in a bad order), and path compression only fixes a path after you've already paid to walk its full length once — it doesn't prevent a single expensive find before that payoff kicks in."
  - q: "Why is union-find the right tool for Kruskal's algorithm specifically, rather than just rerunning a reachability check per edge?"
    a: "Kruskal's processes edges in increasing weight order and needs to repeatedly ask 'are these two endpoints already connected?' across an evolving set of unions; DSU answers that in near-constant time per edge via find(), whereas rerunning BFS/DFS for every candidate edge would cost O(V+E) each time."
deeper:
  - label: "Disjoint Set Union — CP-Algorithms reference"
    url: "https://cp-algorithms.com/data_structures/disjoint_set_union.html"
related_formulas: []
---

## The problem DSU solves

You have a collection of elements, and the only question you ever need to ask is "are these two elements in the same group?" and "merge these two groups into one." You never need to ask "list everyone in this group" or "remove this element from its group." That narrow contract is exactly what disjoint-set-union (DSU), also called union-find, is built for — and because the contract is narrow, the implementation can be startlingly fast.

The structure represents each group as a tree, not by storing children, but by storing one parent pointer per node. The root of a tree is the group's representative: two elements are in the same group exactly when following parent pointers from each of them lands on the same root. `find(x)` walks up from `x` to its root. `union(x, y)` finds the roots of `x` and `y` and attaches one root under the other.

## Why a naive version is slow

If `union` always attaches whichever root it's handed as the child of the other (say, always make the second root a child of the first), you can construct a worst case where the tree becomes a straight chain: union(1,2), union(2,3), union(3,4), ... Each `find` on the far end then costs O(n). That's no better than a linked list.

## Union by rank (or size): bounding the height

The fix is to never let a union make the tree taller than necessary. Track a "rank" per root — an upper bound on that tree's height — and when unioning two roots, attach the one with smaller rank under the one with larger rank. Only when ranks are equal does the new root's rank increase by one. Attaching the shallow tree under the deep one (rather than the reverse) never increases the resulting tree's height by more than what's already implied by the ranks. A clean inductive argument shows a tree with rank r must have at least 2^r nodes, so with n elements, rank — and hence worst-case find cost — is bounded by O(log n). Union-by-size (track node counts instead of heights, attach the smaller-count tree under the larger) gives the same asymptotic guarantee via a similar argument.

## Path compression: flattening on the way up

Union by rank caps the height at O(log n), but it doesn't shrink trees that are already that tall. Path compression does: every time `find(x)` walks up to the root, re-point every node it passed through directly to that root before returning. The next `find` on any of those nodes is now O(1).

Trace an example: suppose rank-based unions have built a chain-like tree 1 -> 2 -> 3 -> 4 -> 5 (5 is root). Calling `find(1)` walks 1 -> 2 -> 3 -> 4 -> 5, discovers the root is 5, and on the way back re-parents 1, 2, 3, and 4 all directly to 5. The tree is now flat: five nodes, all children of 5. Any future `find` on 1 through 4 is a single hop.

## Why the combination is what gives near-O(1)

Each technique alone gets you to O(log n). The reason the pairing collapses to the inverse Ackermann function, O(alpha(n)), is subtler: union-by-rank guarantees no tree ever gets *pathologically* tall to begin with, which means path compression's flattening work on any one find is bounded, and that flattening permanently benefits every future find through those nodes. The formal amortized analysis (due to Tarjan) shows that over any sequence of m operations on n elements, the total cost is O(m \* alpha(n)). Since alpha(n) — the inverse of the famously explosive Ackermann function — is under 5 for any n you could physically represent, this is "constant time" in every practical sense, even though it isn't technically O(1) in the strict asymptotic sense.

## Real uses beyond the textbook demo

Kruskal's MST algorithm sorts edges by weight and adds each edge unless its endpoints are already unioned — DSU answers "already connected?" in near-O(1) instead of rerunning a traversal per edge. Cycle detection in an undirected graph is the same check: an edge connecting two already-unioned vertices closes a cycle. And DSU is the standard structure for *online* (incremental) connected-components queries — when edges only ever get added over time, never removed, DSU answers "how many components right now?" and "are u and v connected?" far faster than recomputing components from scratch after every edge.
