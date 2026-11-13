---
title: "Eulerian Paths & Circuits"
description: Visiting every edge of a graph exactly once, the odd-degree-vertex test for when it's possible, and Hierholzer's algorithm for building one.
roadmap: datastructures-and-algorithms
node: eulerian-paths-and-circuits
group: more-graph-algorithms
video:
  provider: youtube
  id: "8MpoO2zA2l4"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 934
level: intermediate
prerequisites: ["graph-traversal-bfs-and-dfs"]
takeaways:
  - "An Eulerian path visits every EDGE of a graph exactly once (vertices may repeat), which is a completely different goal from a Hamiltonian path, which visits every VERTEX exactly once and is NP-hard to find in general."
  - "An undirected connected graph has an Eulerian circuit (a closed Eulerian path) if and only if every vertex has even degree, and it has a non-circuit Eulerian path if and only if exactly two vertices have odd degree."
  - "The odd-degree condition falls directly out of counting: every time a path passes through a vertex without starting or ending there, it uses up exactly two of that vertex's incident edges, so a vertex with odd degree can only be a path's start or its end."
  - "Hierholzer's algorithm builds an Eulerian circuit by walking until it gets stuck back at the start vertex, then splicing in a new sub-circuit at any vertex along that walk that still has unused edges, repeating until no edges remain."
  - "Hierholzer's algorithm runs in O(E) time because each edge is traversed exactly once across all the splicing steps, making it far faster than naively searching for a valid edge ordering."
check_questions:
  - q: "Why can a graph with four odd-degree vertices never have a single Eulerian path, no matter how the edges are arranged?"
    a: "Every vertex the path merely passes through uses two of its edges (one in, one out), so only the two vertices where the path starts and ends are allowed to have an odd number of edges used an odd number of times. With four odd-degree vertices, at least two of them would have to be interior to the path, which is impossible since interior visits always consume an even number of edges at that vertex."
  - q: "Hierholzer's algorithm finds a dead-end sub-circuit before the full Eulerian circuit is complete. Why does splicing that sub-circuit into the main circuit at the shared vertex still produce a valid Eulerian circuit?"
    a: "The shared vertex lies on both circuits, so you can traverse the main circuit up to that vertex, detour through the entire sub-circuit (which starts and ends at the same vertex), and then continue along the rest of the main circuit — every edge in both circuits still gets visited exactly once, and the result is still a single closed walk."
  - q: "Why does Hierholzer's algorithm run in O(E) time instead of something worse, given that it sometimes has to restart and splice in new sub-circuits?"
    a: "Although the algorithm may perform several splicing passes, every edge is removed from the graph (or marked used) the first and only time it is traversed, across all passes combined — the total work done across every sub-circuit never exceeds one pass over every edge."
deeper:
  - label: "Eulerian Path — CP-Algorithms"
    url: "https://cp-algorithms.com/graph/euler_path.html"
related_formulas: []
---

## Every edge, not every vertex

It's easy to conflate an Eulerian path with a Hamiltonian path because both sound like "visit everything once," but they target completely different objects. An **Eulerian path** visits every *edge* of a graph exactly once — vertices can be revisited as many times as needed, since every visit to a vertex (other than the start and end) just means passing through on the way to the next unused edge. A **Hamiltonian path** visits every *vertex* exactly once and doesn't care about edges at all. This distinction matters practically: Eulerian paths can be found or ruled out in linear time, while deciding whether a Hamiltonian path exists is NP-complete. The Eulerian problem's tractability comes from a simple local counting argument, not a search.

## The degree test

Think about what happens to a vertex `v` every time an Eulerian path passes through it without starting or ending there: the path arrives along one edge and leaves along another, consuming exactly two of `v`'s incident edges per pass-through. If `v` has even degree, all of its edges can be paired up this way with none left over. If `v` has odd degree, one edge is always left dangling — meaning the path must either start or end at `v`, since that's the only way to use an odd number of edges at a single vertex (the path uses one edge to "enter" permanently without a matching "exit," or vice versa).

This gives the full classification for a connected undirected graph:
- **Eulerian circuit** (starts and ends at the same vertex) exists iff every vertex has even degree.
- **Eulerian path** (not necessarily closed) exists iff exactly zero or two vertices have odd degree. With zero, you get a circuit; with exactly two, every valid path must start at one of them and end at the other.
- **No Eulerian path** exists if more than two vertices have odd degree — there's no way to avoid leaving multiple dangling edges.

(Directed graphs have an analogous rule: a circuit exists iff in-degree equals out-degree at every vertex, and a path exists iff at most one vertex has out-degree one more than in-degree, at most one has in-degree one more than out-degree, and all others balance.)

## Hierholzer's algorithm, worked through

Knowing a circuit exists doesn't hand you one — that's what **Hierholzer's algorithm** does, in O(E) time. The idea: start at any vertex and walk, always leaving through an unused edge, deleting each edge as you use it. Because every vertex has even degree, you can never get stuck anywhere except back at your starting vertex — every other vertex you enter still has an unused edge to leave by, since you've used an even number of its edges so far. Eventually you return to the start with a closed sub-circuit, but possibly with unused edges left elsewhere in the graph.

Here's the fix: walk along your sub-circuit and find any vertex on it that still has unused edges. Start a *new* walk from there, which — by the same even-degree argument — must also return to that same vertex, forming another closed sub-circuit. Splice this new loop into the original circuit at that vertex (visit the main circuit up to that point, detour through the whole new loop, then resume the main circuit). Repeat this process — find a vertex with leftover edges, walk a new loop from it, splice it in — until no edges remain anywhere. The result is one single Eulerian circuit threading through every edge exactly once.

Concretely: suppose a graph is a square A-B-C-D-A, plus a smaller triangle B-E-F-B attached at B. Starting at A, you might walk A→B→C→D→A — stuck, since A's edges are used up, but B still has its triangle edges unused. Walk the triangle from B: B→E→F→B. Splice it into the first circuit at B: A→B→E→F→B→C→D→A. Every edge used exactly once — done.

For an Eulerian *path* (two odd-degree vertices, not a circuit), the same algorithm works unchanged except you start at one of the two odd-degree vertices; the walk then naturally terminates at the other one instead of looping back, because that vertex is exactly where the odd edge count runs out.
