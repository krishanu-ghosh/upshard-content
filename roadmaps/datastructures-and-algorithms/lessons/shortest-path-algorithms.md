---
title: "Shortest Paths: Dijkstra, Bellman-Ford, A*"
description: Three ways to find the cheapest path through a weighted graph, and why you can't always reach for the simplest one.
roadmap: datastructures-and-algorithms
node: shortest-path-algorithms
group: graph-algorithms
video:
  provider: youtube
  id: "pSqmAO-m7Lk"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 600
level: intermediate
prerequisites:
  - graph-traversal-bfs-and-dfs
takeaways:
  - Dijkstra's algorithm greedily expands the closest unvisited node first, using a priority queue, and finds the shortest path from a source to every other node — but only when every edge weight is non-negative.
  - Bellman-Ford relaxes every edge V-1 times instead of using a priority queue, which makes it slower than Dijkstra but lets it handle negative edge weights, and it can detect a negative-weight cycle as a side effect.
  - A* is Dijkstra with a heuristic bolted on — it still explores by cost-so-far, but adds an estimate of the remaining distance to the goal, which lets it ignore large parts of the graph that are clearly going the wrong way.
  - A* only stays correct if its heuristic never overestimates the true remaining distance, a property called admissibility; otherwise it can return a path that isn't actually shortest.
  - All three algorithms solve the same underlying problem, and the choice between them is really a question about what the graph's edge weights look like and whether you have a decent heuristic.
check_questions:
  - q: Why can't Dijkstra's algorithm handle negative edge weights?
    a: Because Dijkstra finalizes a node's shortest distance the moment it's dequeued, assuming no later discovery could possibly improve it, and a negative edge can violate that assumption by making a longer-looking path cheaper after the fact.
  - q: What does Bellman-Ford give you that Dijkstra can't?
    a: Support for negative edge weights, and the ability to detect a negative-weight cycle by checking whether any edge can still be relaxed after V-1 rounds.
  - q: What property must an A* heuristic have to guarantee a correct shortest path?
    a: It must be admissible — it must never overestimate the true remaining cost to the goal — otherwise A* can settle on a path that looks cheapest under the heuristic but isn't actually the shortest.
deeper:
  - label: "Bellman Ford Algorithm | Shortest path & Negative cycles | Graph Theory"
    url: "https://www.youtube.com/watch?v=lyw4FaxrwHg"
  - label: "A* (A Star) Search Algorithm - Computerphile"
    url: "https://www.youtube.com/watch?v=ySN5Wnu88nE"
related_formulas: []
---

## The same problem, three situations

"Find the cheapest path from A to B" sounds like one problem, but the right algorithm depends on what the graph's weights look like, and whether you have any extra information about where the goal is. Dijkstra's algorithm, Bellman-Ford, and A* search all solve shortest-path problems, and the differences between them are really differences in what assumptions each one is allowed to make.

## Dijkstra: greedy expansion by a priority queue

Dijkstra's algorithm keeps a running "best distance so far" for every node, starting at 0 for the source and infinity for everything else, plus a priority queue ordered by that distance. It repeatedly pulls the closest unvisited node, finalizes its distance, and relaxes its edges — meaning, for each neighbor, it checks whether going through the just-finalized node beats the neighbor's current best distance, and updates it if so.

Trace it on a tiny weighted graph: A→B (4), A→C (1), C→B (1), B→D (1). Starting from A: distances are A=0, everything else infinite. Pop A, relax B (4) and C (1). Pop C (distance 1, the cheapest left), relax B via C: 1 + 1 = 2, which beats B's current 4, so update B to 2. Pop B (now 2), relax D: 2 + 1 = 3. Pop D (3). Final distances: A=0, C=1, B=2, D=3 — notice that going through C turned out cheaper than A's direct edge to B, which is the relaxation step doing its job.

The reason Dijkstra works is that once a node is popped from the priority queue, nothing left in the graph can possibly make its distance smaller, because everything left is at least as far away. That reasoning breaks completely the moment an edge can be negative: a path that looks longer right now could become shorter later by crossing a negative edge, and Dijkstra has already closed the book on the node by then.

## Bellman-Ford: slower, but handles negative weights

Bellman-Ford drops the priority queue and just relaxes every edge in the graph, in any order, repeated V-1 times, where V is the number of vertices — enough rounds to guarantee that the shortest path to any node, which uses at most V-1 edges, has fully propagated. That's O(V·E), noticeably slower than Dijkstra's O((V+E) log V) with a binary heap, but it works correctly even when some edges are negative.

It has a second use: run one more round of relaxation after the V-1 rounds. If any edge can still be relaxed — if some distance can still get smaller — the graph contains a negative-weight cycle, meaning there is no well-defined shortest path at all, because you could loop around that cycle forever and keep shaving cost off. Detecting that is something Dijkstra simply can't do.

## A*: Dijkstra with a sense of direction

A* search is Dijkstra plus a heuristic function h(n) that estimates the remaining distance from node n to the goal. Instead of ordering the priority queue purely by cost-so-far g(n), A* orders it by f(n) = g(n) + h(n) — actual cost so far, plus estimated cost to go. If you're navigating a road network and the goal is due east, a straight-line-distance heuristic lets A* deprioritize exploring nodes to the west almost entirely, often visiting a small fraction of the nodes Dijkstra would.

The catch is that A* is only guaranteed to find the true shortest path if the heuristic is admissible — it must never overestimate the real remaining distance. Straight-line distance is admissible for road networks because you can never actually get somewhere faster than a straight line. A heuristic that overestimates can cause A* to deprioritize a node that was actually on the true shortest path, and return a path that only looks optimal under the heuristic.

## Picking one

If every edge weight is non-negative and you need distances from one source to everywhere: Dijkstra. If edges can be negative, or you need to detect a negative cycle: Bellman-Ford. If you have a single source and a single goal, and a decent admissible estimate of remaining distance: A*, which can be dramatically faster in practice by refusing to waste time exploring in the wrong direction.
