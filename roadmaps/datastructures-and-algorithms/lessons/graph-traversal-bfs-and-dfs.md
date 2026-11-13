---
title: "Graph Traversal: BFS and DFS"
description: Breadth-first and depth-first search explore a graph in opposite orders — what each visits first, and when that difference matters.
roadmap: datastructures-and-algorithms
node: graph-traversal-bfs-and-dfs
group: graph-algorithms
video:
  provider: youtube
  id: "oDqjPvD54Ss"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 600
level: beginner
prerequisites:
  - graph-representation
takeaways:
  - Breadth-first search visits a graph in widening rings around the start node, which is exactly what you need to find the shortest path in an unweighted graph.
  - Depth-first search commits to one path as far as it can go before backing up, which makes it the natural choice for exploring every reachable node, detecting cycles, or finding connected components.
  - BFS is implemented with a queue and an explicit "visited" set; DFS is implemented with a stack, or just as often with recursion, which is a stack in disguise.
  - Both algorithms visit every vertex and every edge at most once, so both run in O(V + E) time regardless of how the graph is shaped.
  - The "right" traversal is a question about what you're trying to learn from the graph, not a question of which algorithm is generally better.
check_questions:
  - q: Why does BFS, rather than DFS, guarantee the shortest path in an unweighted graph?
    a: Because BFS explores all nodes at distance k before any node at distance k+1, so the first time it reaches a node is guaranteed to be via a shortest path; DFS can wander down a long path before trying a shorter one.
  - q: What's the practical relationship between recursive DFS and an explicit stack?
    a: Recursive DFS uses the call stack to remember which node to backtrack to, so an explicit-stack version of DFS does the same bookkeeping by hand instead of relying on the language runtime.
  - q: Both BFS and DFS run in O(V + E) — so why does the choice between them matter at all?
    a: Because they visit nodes in a different order, which matters for what you can conclude along the way — BFS's order gives you shortest paths for free, DFS's order gives you natural recursion into subtrees and easy cycle detection.
deeper:
  - label: "Depth First Search Algorithm"
    url: "https://www.youtube.com/watch?v=7fujbpJ0LB4"
related_formulas: []
---

## Two ways to explore the same graph

Given a graph and a starting vertex, there are exactly two sane orders to visit everything reachable from it: go wide, or go deep. Breadth-first search (BFS) goes wide — it visits every neighbor of the start, then every neighbor of those neighbors, expanding outward in rings. Depth-first search (DFS) goes deep — it picks a neighbor, dives into its subtree completely, and only backs up when it hits a dead end.

Neither order is "the" way to traverse a graph; they're different lenses, and which one you want depends on the question you're asking.

## BFS: a queue and expanding rings

BFS keeps a queue of nodes to visit and a set of nodes already seen. It starts by enqueuing the source node and marking it visited. Then, until the queue is empty, it dequeues a node, looks at all of its unvisited neighbors, marks each one visited, and enqueues it.

Trace it on a small graph: A connects to B and C; B connects to D; C connects to D and E. Starting from A:

- Visit A, enqueue B and C. Queue: [B, C]
- Dequeue B, visit its neighbor D (new), enqueue it. Queue: [C, D]
- Dequeue C, its neighbor D is already visited, enqueue E. Queue: [D, E]
- Dequeue D, then E — nothing new left.

The visit order is A, B, C, D, E — A's distance-1 neighbors first (B, C), then its distance-2 neighbors (D, E). That ring-by-ring property is BFS's whole value: the first time you reach any node, you've reached it by the fewest possible edges, which makes BFS the standard way to compute shortest paths when every edge has the same weight.

## DFS: a stack, or recursion

DFS keeps a stack instead of a queue — or, more commonly, just recurses, letting the call stack do the bookkeeping. From a node, it picks an unvisited neighbor, recurses into it immediately, and only returns to try the next neighbor once that whole branch is exhausted.

On the same graph, starting from A: visit A, go to B (first neighbor), from B go to D (its only new neighbor), from D there's nothing new, so backtrack to B (done), backtrack to A, go to C (second neighbor), from C go to E (D is already visited). Order: A, B, D, C, E — a very different order from BFS's A, B, C, D, E, even though both visited every node exactly once.

DFS's "commit, then backtrack" structure is what makes it natural for problems that are themselves recursive: exploring a maze, listing every node of a tree via pre/in/post-order, detecting a cycle (you've found one the moment you reach a node that's already on your current path, not just visited at all), or finding connected components (run DFS from every unvisited node; each run finds one component).

## Choosing between them

Both touch every vertex once and every edge once, so both cost O(V + E) — the same asymptotic bill. The decision is about what you need the traversal to hand you:

- Need the shortest path, or "closest" nodes first? BFS.
- Need to explore a whole subtree, detect a cycle, or do anything naturally recursive? DFS.
- Working with a huge, very wide graph (like a social network) where BFS's queue could balloon in size? DFS's stack depth is bounded by the graph's depth, which is sometimes the more memory-friendly choice — though on a very deep graph, recursive DFS can blow the call stack, which is exactly why an explicit-stack version exists.

Memorize the mechanism, not just the name: a queue gives you rings outward from the start; a stack (or recursion) gives you full commitment to one path before trying the next. Everything else about BFS and DFS — their use cases, their implementations, their failure modes — follows from that one difference.
