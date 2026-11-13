---
title: "Topological Sorting"
description: Ordering a DAG so every dependency comes before what depends on it — two different algorithms reach the same ordering from opposite directions.
roadmap: datastructures-and-algorithms
node: topological-sorting
group: union-find-and-graph-structure
video:
  provider: youtube
  id: "cIBFEhD77b4"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 812
level: intermediate
prerequisites: []
takeaways:
  - "A topological order only exists for a directed acyclic graph (DAG); a cycle makes it impossible to put every edge's tail before its head, since you'd eventually have to put some node before itself."
  - "Kahn's algorithm builds the order forward: repeatedly peel off a node with in-degree zero, 'remove' its outgoing edges by decrementing its neighbors' in-degrees, and queue any neighbor that just hit zero."
  - "The DFS-based approach builds the order backward: run DFS, append each node to a list only after all of its descendants have been fully explored (postorder), then reverse that list."
  - "If Kahn's algorithm finishes with fewer nodes processed than the graph has, the unprocessed remainder is necessarily involved in a cycle — this is a free, built-in cycle detector."
  - "Both algorithms run in O(V + E), visit every vertex and edge exactly once, and can produce different valid orderings for the same DAG — topological order is not unique when ties exist."
check_questions:
  - q: "Why does DFS have to append a node to the output AFTER recursing into its children, rather than before, for the reversed result to be a valid topological order?"
    a: "A node must come before everything it points to in the final order. If you append on entry (preorder) and then reverse, a node could still end up after something it points to if that neighbor was visited via a different branch; appending only once all descendants are fully processed (postorder) guarantees everything reachable from a node is already in the list before the node itself is added, so reversing puts the node first."
  - q: "Why does Kahn's algorithm fail to produce a full ordering on a graph with a cycle, and how does that show up concretely?"
    a: "Every node inside a cycle always has at least one incoming edge from another node in that same cycle, so its in-degree never drops to zero no matter what else gets processed — those nodes simply never enter the queue, so the algorithm terminates having output fewer nodes than the graph contains."
  - q: "Given a DAG with multiple nodes that have in-degree zero at the same time, does Kahn's algorithm produce a unique topological order?"
    a: "No — whichever zero-in-degree node is dequeued first is an arbitrary choice (often decided by queue order), and different valid choices there produce different, equally correct topological orderings; uniqueness only happens when the DAG has a Hamiltonian path, forcing one ordering at each step."
deeper:
  - label: "Topological Sort Algorithm (DFS-based) | Graph Theory"
    url: "https://www.youtube.com/watch?v=eL-KzMXSXXI"
related_formulas: []
---

## What ordering means here

A topological order of a directed graph is a linear arrangement of its vertices such that for every directed edge u -> v, u appears before v. Think of each edge as "u must happen before v" — a build dependency, a course prerequisite, a step in a pipeline. A topological order is a valid schedule that respects every one of those constraints simultaneously.

This only works if the graph is a DAG (directed, acyclic). If there's a cycle A -> B -> C -> A, no linear order can place A before B, B before C, *and* C before A all at once — the constraints are contradictory. So the existence of a topological order and the absence of a cycle are the same fact viewed two ways, which is why both algorithms below double as cycle detectors.

## Kahn's algorithm: peel from the sources

Kahn's algorithm thinks in terms of in-degree: the number of edges pointing *into* each node. A node with in-degree zero has no unresolved dependency, so it's safe to place next in the order right now.

The mechanics: compute in-degree for every vertex, and put every vertex with in-degree zero into a queue. Then, until the queue is empty, dequeue a vertex, append it to the output, and for each of its outgoing edges, decrement the neighbor's in-degree — effectively removing that edge from consideration. Any neighbor whose in-degree just hit zero goes into the queue.

Worked example: vertices A, B, C, D with edges A->C, B->C, C->D. In-degrees: A=0, B=0, C=2, D=1. Queue starts with [A, B] (both sources). Dequeue A, output [A], decrement C to 1 (not zero yet). Dequeue B, output [A, B], decrement C to 0 — enqueue C. Dequeue C, output [A, B, C], decrement D to 0 — enqueue D. Dequeue D, output [A, B, C, D]. Every edge points forward in that final list.

If the queue ever empties while vertices remain unprocessed, those leftover vertices form (or are entangled in) a cycle — none of them ever reached in-degree zero, meaning each is permanently waiting on something that's waiting on it.

## The DFS-based approach: finish last, appear first

The DFS method flips the direction of reasoning. Run a standard DFS from every unvisited vertex. The key rule: a vertex is only added to a result list once the DFS call on it has fully returned — that is, after every vertex reachable from it has already been explored and added. This produces a *postorder* list. Reverse that list, and it's a valid topological order.

Why reversing postorder works: if there's an edge u -> v, then the DFS call on u must recurse into v (directly or indirectly) before u's own call can finish — so v finishes (and gets appended) strictly before u does, in postorder. Reversing the list then puts u before v, exactly as required.

Using the same example graph: DFS from A visits A -> C -> D; D has no outgoing edges so it finishes first and is appended: [D]. C then finishes: [D, C]. A finishes: [D, C, A]. DFS from B (still unvisited) visits B -> C, but C is already visited, so B finishes immediately: [D, C, A, B]. Reversed: [B, A, C, D] — another valid ordering (notice it differs from Kahn's [A, B, C, D], since both A and B are legitimately interchangeable at the front).

## Choosing between them

They're asymptotically identical, O(V + E), and both detect cycles as a side effect — Kahn's by leftover unprocessed nodes, DFS-based by detecting a "back edge" (an edge into a node currently on the recursion stack) during the traversal. Kahn's is often preferred when you want an explicit, iterative, BFS-style implementation or need to process nodes level-by-level (e.g., parallel task scheduling, where everything at in-degree zero at once can run concurrently). The DFS-based version is often more natural when topological sort is a small step inside a larger DFS-based algorithm, such as computing strongly connected components.
