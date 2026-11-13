---
title: "Articulation Points & Bridges"
description: Finding the single vertices and edges whose removal disconnects a graph, using one DFS pass that tracks discovery time and low-link values.
roadmap: datastructures-and-algorithms
node: articulation-points-and-bridges
group: more-graph-algorithms
video:
  provider: youtube
  id: "qrAub5z8FeA"
  start: 0
  end: null
creator:
  name: take U forward
  channel_url: https://www.youtube.com/@takeUforward
duration_seconds: 1404
level: intermediate
prerequisites: ["graph-traversal-bfs-and-dfs"]
takeaways:
  - "A bridge is an edge, and an articulation point is a vertex, whose removal increases the number of connected components in the graph — both represent single points of failure in a network."
  - "Tarjan's technique finds both in a single DFS pass by tracking, for every vertex, its discovery time (when DFS first reaches it) and its low-link value (the earliest discovery time reachable from its subtree using at most one back edge)."
  - "A back edge — one that connects a vertex to an already-visited ancestor, not a new vertex — is exactly what lets low-link values 'see past' the current vertex, since it proves an alternate route exists that bypasses it."
  - "A tree edge (u, v), where v is u's DFS child, is a bridge exactly when low[v] > disc[u] — meaning v's subtree has no back edge reaching u or anything above u, so that tree edge is the subtree's only connection to the rest of the graph."
  - "A non-root vertex u is an articulation point if some child v has low[v] >= disc[u] (v's subtree can't escape past u even by reaching u itself); the DFS root is a special case, an articulation point only if it has more than one child in the DFS tree."
check_questions:
  - q: "Why does the bridge condition use a strict inequality (low[v] > disc[u]) while the articulation point condition for non-root vertices uses a non-strict one (low[v] >= disc[u])?"
    a: "A bridge requires v's subtree to have absolutely no back edge reaching u or higher — if it could reach exactly u (low[v] == disc[u]), the edge (u,v) still isn't a bridge because u itself remains a shared connection point, but u still becomes a cut vertex since removing u would strand that subtree from everything above it. The articulation point test only needs v's subtree to be unable to bypass u, which includes reaching u itself, hence the non-strict comparison."
  - q: "Why does the DFS root need a special-case rule (articulation point iff it has 2+ DFS children) instead of using the same low-link comparison as every other vertex?"
    a: "The low-link comparison checks whether a subtree can reach back past its parent toward the rest of the graph, but the root has no parent to reach past — there's nothing 'above' it for a back edge to connect to. Instead, the root is only critical if removing it disconnects two of its own subtrees from each other, which is exactly what having two or more separate DFS children means, since there was no edge between those subtrees other than through the root."
  - q: "In a graph that's just one long cycle (every vertex has degree 2), DFS from any vertex finds no articulation points and no bridges. Why does the low-link computation produce that result?"
    a: "DFS walks around the cycle as a single path of tree edges, and the one edge left over closes the loop back to the start vertex — a back edge from the last vertex straight to the root. That back edge gives every vertex on the path a low-link value equal to the root's discovery time (0), since each vertex can reach that back edge through its descendants, so low[v] is never greater than or equal to its parent's discovery time except trivially at points the back edge doesn't cover — and in a simple cycle it covers the whole thing, so no tree edge qualifies as a bridge and no interior vertex qualifies as an articulation point."
deeper:
  - label: "Find Articulation Points using Tarjan's Algorithm — Techdose"
    url: "https://www.youtube.com/watch?v=64KK9K4RpKE"
related_formulas: []
---

## Two kinds of single point of failure

A network can fail at a vertex or at an edge. A **bridge** is an edge whose removal splits the graph into more connected components than before — it's the only link between two otherwise-separate regions. An **articulation point** (or cut vertex) is a vertex with the same property: deleting it (and every edge touching it) increases the component count. Both describe fragility, and both can be found together in one linear-time DFS pass, because they share the same underlying question: "does this part of the graph have any way back to the rest, other than through this one edge or vertex?"

## Discovery time and low-link value

Run a DFS and assign every vertex a **discovery time**, `disc[v]` — a counter that increments each time DFS visits a new vertex, so it records the order vertices were first reached. As DFS proceeds, some edges become **tree edges** (leading to an unvisited vertex) and others become **back edges** (leading to an already-visited ancestor still on the current DFS stack). A back edge is the key structural clue here: it's a "shortcut" that proves part of the graph can reach further up the DFS tree without going through the current vertex's parent.

For every vertex `v`, define its **low-link value**, `low[v]`, as the smallest discovery time reachable from `v`'s subtree using at most one back edge — formally, the minimum of `disc[v]` itself, the low-link values of all of `v`'s DFS children, and the discovery time of any vertex reachable from `v` by a direct back edge. Intuitively, `low[v]` answers: "if I stood at `v` and could use the tree edges below me plus exactly one back edge, how far back up the tree could I climb?" Both values are computed during a single post-order step as DFS returns from each child.

## Reading off bridges and articulation points

With `disc` and `low` computed, the two conditions fall out directly:

**Bridge test:** a tree edge `(u, v)`, where `v` is `u`'s DFS child, is a bridge exactly when `low[v] > disc[u]`. This says `v`'s entire subtree has no back edge reaching `u` or any ancestor of `u` — the only way into or out of that subtree is through the single edge `(u, v)`. If instead `low[v] <= disc[u]`, some descendant of `v` has a back edge reaching `u` or higher, giving the subtree an alternate escape route, so `(u, v)` isn't essential.

**Articulation point test:** a non-root vertex `u` is an articulation point if any DFS child `v` has `low[v] >= disc[u]`. This is the weaker, non-strict version of the bridge condition — `v`'s subtree being unable to reach *past* `u` (even if it can reach `u` itself) still means removing `u` strands that subtree from the rest of the graph above it. The DFS root is handled separately: it has no parent to "reach past," so instead it's an articulation point exactly when it has two or more separate DFS children, since that means those children's subtrees had no edge connecting them except through the root.

## Worked example

Take vertices 1–2–3 forming a triangle, with 3–4 and 4–5 as a trailing path. DFS from 1: visit 1 (disc=0), 2 (disc=1), 3 (disc=2) — and 3 has a back edge to 1, so `low[3] = 0`. That back edge also pulls `low[2] = 0` (since 2's child is 3) and `low[1] = 0`. Continuing from 3, DFS visits 4 (disc=3, no back edges, so `low[4] = 3`) and then 5 (disc=4, `low[5] = 4`).

Check edge (3,4): `low[4] = 3`, `disc[3] = 2`. Since `3 > 2`, this is a bridge — correct, since removing it strands vertex 5 and 4. Check edge (4,5): `low[5] = 4 > disc[4] = 3`, also a bridge. Check edge (1,2) or (2,3): their children's low-link values are all 0, never exceeding the parent's discovery time, so neither is a bridge — consistent with the triangle having a redundant path between any two of its vertices. For articulation points: vertex 3 has child 4 with `low[4] = 3 >= disc[3] = 2`, so 3 is an articulation point (removing it strands 4 and 5). Vertex 4 similarly is one (its child 5 has `low[5] = 4 >= disc[4] = 3`). Vertices 1 and 2 are not, since the triangle gives every pair of them a second route.
