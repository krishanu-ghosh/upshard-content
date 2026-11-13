---
title: "Minimum Spanning Trees: Prim's and Kruskal's"
description: Connecting every node in a weighted graph as cheaply as possible, two different ways — grow a tree, or sort the edges.
roadmap: datastructures-and-algorithms
node: minimum-spanning-trees
group: graph-algorithms
video:
  provider: youtube
  id: "xq3ABa-px_g"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 900
level: intermediate
prerequisites:
  - shortest-path-algorithms
takeaways:
  - A minimum spanning tree connects every vertex in a weighted, connected, undirected graph using the fewest and cheapest possible edges, with no cycles.
  - Prim's algorithm grows a single tree outward, always adding the cheapest edge that connects a node already in the tree to one outside it.
  - Kruskal's algorithm ignores the tree's shape entirely — it sorts all edges by weight and adds each one, cheapest first, as long as it doesn't create a cycle.
  - Kruskal's needs a union-find (disjoint-set) structure to detect cycles quickly, since checking "would this edge create a cycle?" the naive way would be far too slow on a large graph.
  - Both algorithms are greedy — they never reconsider a choice — and both are provably optimal for this problem, a rarer guarantee than greedy algorithms usually get.
check_questions:
  - q: What's the core difference in how Prim's and Kruskal's build the minimum spanning tree?
    a: Prim's grows one connected tree outward one edge at a time, always picking the cheapest edge touching the current tree; Kruskal's considers all edges globally, sorted by weight, and adds any edge that doesn't create a cycle, regardless of which tree it touches.
  - q: Why does Kruskal's algorithm need a union-find structure?
    a: To quickly check whether the two endpoints of a candidate edge are already connected before adding it — without that check, detecting a cycle by searching the whole tree for every edge would be far too slow.
  - q: Is it possible for a graph to have more than one minimum spanning tree?
    a: Yes — if two different edges share the same weight, swapping one for the other can produce a different tree with the exact same total weight, so the minimum spanning tree is optimal in total cost but not always unique.
deeper:
  - label: "Kruskal's Algorithm Visually Explained (Minimum Spanning Tree)"
    url: "https://www.youtube.com/watch?v=OxfTT8slSLs"
related_formulas: []
---

## Connecting everything as cheaply as possible

Given a weighted, connected, undirected graph — think of it as a set of cities and the cost of building a road between each pair — a minimum spanning tree (MST) is the cheapest possible set of roads that still connects every city to every other one, directly or indirectly. "Spanning" means every vertex is included; "tree" means there are no cycles, which is also why an MST on V vertices always has exactly V-1 edges — any fewer and something would be disconnected, any more and you'd have a cycle you could remove an edge from and still be connected, for less money.

There are two classic ways to build one, and they take opposite approaches to the same greedy idea.

## Prim's algorithm: grow one tree

Prim's starts from an arbitrary vertex and grows a single tree outward. At each step, it looks at every edge that connects a vertex already in the tree to a vertex not yet in the tree, picks the cheapest one, and adds it, bringing one new vertex into the tree. Repeat until every vertex is included.

Trace it on four cities A, B, C, D with roads A-B (1), A-C (4), B-C (2), B-D (5), C-D (1). Start the tree at A. The cheapest edge leaving the tree is A-B (1); add it — tree is {A, B}. The cheapest edge leaving {A, B} is B-C (2), beating A-C's 4; add it — tree is {A, B, C}. The cheapest edge leaving {A, B, C} is C-D (1), beating B-D's 5; add it — tree is {A, B, C, D}, done. Total cost: 1 + 2 + 1 = 4.

An efficient implementation keeps a priority queue of "cheapest known edge into the tree" for every vertex not yet included, so each step is a cheap pop-and-update rather than a full rescan of every edge.

## Kruskal's algorithm: sort first, ignore shape

Kruskal's takes a completely different angle: forget about growing one connected shape. Sort every edge in the entire graph by weight, cheapest first, and walk down that sorted list adding each edge, unless adding it would create a cycle, in which case skip it. Stop once you've added V-1 edges.

On the same graph, sorted edges are A-B (1), C-D (1), B-C (2), A-C (4), B-D (5). Add A-B (1), no cycle. Add C-D (1), no cycle — now {A, B} and {C, D} are two separate pieces. Add B-C (2), which connects the two pieces into one, still no cycle. That's 3 edges for 4 vertices — done; A-C and B-D are never even considered. Same total cost, 4, reached by a completely different process, and notice it even started from a different pair of edges than Prim's did, because it isn't tracking "the tree," just "does this edge close a loop."

The cycle check is the one part that needs care: naively, checking "are these two vertices already connected?" by searching the whole structure is slow to repeat for every edge. A union-find (disjoint-set) data structure answers that question in close to constant time, which is why Kruskal's is almost always implemented with one.

## Why either one gives the actual optimum

It's not obvious that greedy, never-look-back choices like these should produce a globally optimal tree — greedy algorithms usually only produce a good-enough answer. Minimum spanning trees are a case where a property called the cut property guarantees greedy correctness: for any way of splitting the graph's vertices into two groups, the cheapest edge crossing that split is always safe to include in some MST. Both Prim's, which repeatedly finds the cheapest edge crossing the tree-versus-not-tree split, and Kruskal's, which processes edges in exactly the order the cut property licenses, are just two different strategies for exploiting that same guarantee.

## Picking one

Prim's, with a priority queue, tends to be faster on dense graphs, where there are lots of edges relative to vertices, since it never has to sort the whole edge list. Kruskal's tends to be simpler to reason about and faster on sparse graphs, where sorting a short edge list is cheap and union-find keeps the cycle check fast. Both are standard enough that either is a safe default; the choice rarely matters outside of genuinely large graphs.
