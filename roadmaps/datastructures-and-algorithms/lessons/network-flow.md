---
title: "Network Flow: Max-Flow and Ford-Fulkerson"
description: How much can move from a source to a sink through a capacity-limited network, and the augmenting-path method that finds out.
roadmap: datastructures-and-algorithms
node: network-flow
group: graph-algorithms
video:
  provider: youtube
  id: "LdOnanfc5TM"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 900
level: advanced
prerequisites:
  - minimum-spanning-trees
takeaways:
  - A flow network is a directed graph where every edge has a capacity, and the max-flow problem asks for the largest amount that can move from a source to a sink without any edge exceeding its capacity.
  - The Ford-Fulkerson method repeatedly finds an augmenting path — a path from source to sink with leftover capacity — and pushes as much flow as that path's bottleneck allows.
  - A residual graph tracks how much more flow each edge could still carry, plus a reverse edge for every forward edge, so the algorithm can undo part of an earlier choice if a better routing appears later.
  - Ford-Fulkerson keeps finding augmenting paths and updating the residual graph until no augmenting path from source to sink exists, at which point the flow found is maximum.
  - The max-flow min-cut theorem guarantees that the maximum flow through a network exactly equals the capacity of the smallest cut that would have to be severed to disconnect the source from the sink.
check_questions:
  - q: What is an augmenting path, and what does Ford-Fulkerson do with one?
    a: A path from source to sink in the residual graph that still has spare capacity on every edge; Ford-Fulkerson pushes flow along it equal to the smallest remaining capacity on the path, then updates the residual graph.
  - q: Why does the residual graph include reverse edges?
    a: So the algorithm can effectively cancel part of an earlier flow assignment if routing flow a different way later turns out to allow more total flow overall — without the reverse edge, an early suboptimal choice could never be corrected.
  - q: What does the max-flow min-cut theorem say?
    a: That the maximum amount of flow possible from source to sink exactly equals the minimum total capacity of edges that would need to be removed to separate the source from the sink.
deeper: []
related_formulas: []
---

## Moving as much as possible through a limited pipe system

Picture a network of pipes: water enters at a single source, exits at a single sink, and every pipe in between can carry only up to its own capacity per unit of time. The max-flow problem asks what's the most water you can push from source to sink without violating any pipe's capacity. The same structure models far more than literal pipes — data through a network, traffic through roads, or any system of capacity-limited routes between a single origin and destination.

## The residual graph: what capacity is left

The key idea behind Ford-Fulkerson is to track, for every edge, how much more flow it could still carry — its residual capacity — and to add a reverse edge for every edge that currently carries flow, with residual capacity equal to however much flow is already on it. That reverse edge is what lets the algorithm change its mind: if sending flow along edge u→v turns out to have been a mistake once more of the graph is explored, the algorithm can effectively send flow back along the reverse edge v→u, undoing part of that earlier choice, without ever literally removing flow that's already been counted.

## Ford-Fulkerson: find a path, push flow, repeat

The method is simple to state: while there exists a path from source to sink in the residual graph with positive capacity on every edge along it — an augmenting path — find one, usually with a BFS or DFS, determine its bottleneck (the smallest residual capacity among its edges), and push that much additional flow along the whole path, updating every edge's residual capacity (decreasing it on the forward direction, increasing it on the reverse direction) as you go. Repeat until no augmenting path exists.

Trace it on a tiny network: source S, sink T, with S→A (capacity 3), S→B (2), A→T (2), A→B (1), B→T (3).

First augmenting path: S→A→T, bottleneck `min(3, 2) = 2`. Push 2 units. S→A now has 1 capacity left, A→T has 0 left, and new reverse edges A→S (2) and T→A (2) exist.

Second augmenting path: S→B→T, bottleneck `min(2, 3) = 2`. Push 2 units. S→B has 0 left, B→T has 1 left.

A third path is still open: S→A→B→T. S→A has 1 unit left, A→B still has its full original 1 unit (nothing has touched it yet), and B→T has 1 unit left — bottleneck `min(1, 1, 1) = 1`. Push that unit. Now every edge leaving S is at 0 residual capacity, so no augmenting path remains.

Total flow pushed: `2 + 2 + 1 = 5`, and that's the maximum. A quick min-cut check confirms it: the cut separating `{S, A, B}` from `{T}` severs A→T and B→T, for a capacity of `2 + 3 = 5` — nothing can flow past that ceiling.

## Why it stops at the right answer

The algorithm terminates exactly when no augmenting path remains, meaning there's no way to route even one more unit from S to T anywhere in the residual graph. At that point, the set of vertices still reachable from S in the residual graph, versus the set that isn't, defines a cut: a split of the graph into two sides such that every edge crossing from the source's side to the sink's side is at full capacity, otherwise it would be part of an augmenting path. The max-flow min-cut theorem says this is no coincidence — the maximum possible flow always exactly equals the minimum total capacity of any cut separating source from sink, because that cut's capacity is a hard ceiling nothing can flow past, and Ford-Fulkerson's termination condition is exactly the moment it has found a flow that saturates one.

## Where it matters

Beyond literal network capacity planning, max-flow models bipartite matching (matching applicants to jobs, with edge capacities of 1), image segmentation, airline scheduling, and project selection problems — any time a problem can be rephrased as how much can move from one designated point to another, through capacity-limited channels. Recognizing that disguise is usually the hard part; once a problem is phrased as a flow network, Ford-Fulkerson, or its faster refinements like Edmonds-Karp, which picks augmenting paths via BFS to guarantee polynomial time, handles the rest.
