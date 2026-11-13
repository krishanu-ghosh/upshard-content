---
title: "Strongly Connected Components"
description: Finding the maximal clusters of a directed graph where every node can reach every other — and collapsing them into a clean DAG.
roadmap: datastructures-and-algorithms
node: strongly-connected-components
group: union-find-and-graph-structure
video:
  provider: youtube
  id: "wUgWX0nc4NY"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 1061
level: intermediate
prerequisites:
  - topological-sorting
takeaways:
  - "A strongly connected component (SCC) is a maximal set of vertices where every vertex can reach every other vertex via directed edges — maximal meaning you can't add another vertex without breaking that property."
  - "Tarjan's algorithm finds all SCCs in a single DFS pass by tracking each node's discovery index and its 'low-link' value — the lowest discovery index reachable from it, including through back edges to an ancestor still on the stack."
  - "A node closes off an SCC exactly when its low-link value equals its own discovery index — meaning nothing in its subtree can reach further back up the stack than the node itself."
  - "Kosaraju's algorithm instead runs DFS twice: once to get a finishing-order (like topological sort) on the original graph, then again on the transposed (edge-reversed) graph, processing nodes in reverse finishing order — each DFS tree in the second pass is exactly one SCC."
  - "Contracting every SCC into a single 'super-node' always yields a DAG (the 'condensation'), because any remaining cycle across components would by definition merge them into one larger SCC."
  - "SCCs are the core of 2-SAT solving: build an implication graph of a or b style clauses as directed edges, and the formula is satisfiable exactly when no variable and its negation land in the same SCC."
check_questions:
  - q: "Why does Tarjan's algorithm need a stack of 'currently active' nodes in addition to the usual DFS visited marking?"
    a: "The low-link calculation needs to distinguish a back edge to an ancestor still being processed (which means the two nodes are in the same SCC) from a cross edge to a node that's fully finished and already assigned to some other, earlier-closed SCC; the stack holds exactly the nodes whose SCC hasn't been finalized yet, so an edge only updates low-link if it points to something still on that stack."
  - q: "Why does Kosaraju's algorithm need to run DFS on the transposed graph rather than the original graph in its second pass?"
    a: "The first pass's finishing order puts nodes that can reach a lot of the graph late in the list. Running DFS again on the original graph in that order would just re-explore the same large reachable sets and merge unrelated components; reversing every edge first means a second DFS from a given node can now only reach nodes that could originally reach it, isolating each SCC in reverse-reachability instead of forward-reachability."
  - q: "Why is the condensation graph (SCCs contracted to single nodes) guaranteed to be acyclic?"
    a: "If the condensation had a cycle, it would mean component X can reach component Y and component Y can reach back to component X through directed edges; but then every vertex in X could reach every vertex in Y and vice versa, so X and Y would actually be mutually reachable and therefore part of the same SCC to begin with, contradicting that they were separate components."
deeper:
  - label: "Kosaraju's Algorithm (takeuforward)"
    url: "https://www.youtube.com/watch?v=R6uoSjZ2imo"
related_formulas: []
---

## Mutual reachability, not just reachability

In an undirected graph, "connected component" has one obvious meaning. In a directed graph, reachability isn't symmetric — A might reach B without B being able to reach A — so "connected" splits into two different questions. A strongly connected component (SCC) answers the stricter one: the maximal groups of vertices where every vertex in the group can reach every other vertex in the group, following edge directions both ways around some cycle.

A single vertex with no cycles through it is trivially its own SCC. A graph that's one big cycle is a single SCC. Most real directed graphs are a mix: a few tightly-looped clusters connected by one-way bridges.

## Tarjan's algorithm: one DFS pass, two numbers per node

Tarjan's algorithm computes every SCC in a single depth-first traversal by tracking, for each node, two values: its discovery index (the order it was first visited, assigned once and never changed) and its low-link value (initially equal to its own discovery index, then updated as the DFS explores).

While exploring a node's edges, three cases update the low-link: an edge to an unvisited node recurses first, then takes the minimum of the current low-link and the child's resulting low-link; an edge to a node already on the stack (meaning it's an ancestor in the current DFS tree, not fully finished) takes the minimum of the current low-link and that node's discovery index directly; an edge to a node already visited but no longer on the stack is ignored — it belongs to an already-finalized SCC.

The payoff: after processing all of a node's edges, if its low-link still equals its own discovery index, that node is the "root" of an SCC — nothing in its DFS subtree found a path back further up the stack than this node. At that moment, pop nodes off the stack until you pop this node itself; everything popped is exactly one SCC.

Concretely: in a graph A -> B -> C -> A (a 3-cycle) plus D -> A, DFS visits A (index 0), B (1), C (2). C's edge back to A — still on the stack — pulls C's low-link down to 0. That propagates back: B's low-link becomes 0, A's low-link stays 0. When A finishes, low-link (0) equals its own index (0), so pop C, B, A off the stack as one SCC. D is separate, visited later, with no cycle through it — its own singleton SCC.

## Kosaraju's algorithm: two passes, opposite directions

Kosaraju's trades the bookkeeping of low-links for a second traversal. First, run DFS over the whole graph and record each node's finishing time (same postorder idea as DFS-based topological sort). Second, transpose the graph — reverse every edge — and run DFS again, processing nodes in decreasing order of their first-pass finishing time. Each tree produced by this second DFS is exactly one SCC.

The intuition: a node that finished last in pass one could reach a large part of the graph. Running DFS from it on the *reversed* graph only reaches nodes that could originally reach *it*. The intersection of "can reach it" and "it can reach" is precisely its SCC, and starting from the highest finishing times first guarantees you peel off whole components cleanly without one DFS call accidentally spilling into a different SCC.

## Why this matters: condensation and 2-SAT

Contracting each SCC down to a single super-node always produces a DAG — the "condensation" of the graph. This is a huge simplification: once you know the SCCs, you can run any DAG algorithm (topological sort, longest path, DP) on a much smaller, cycle-free graph instead of the original tangle. It's the standard preprocessing step for analyzing dependency graphs that aren't guaranteed acyclic.

2-SAT is a sharper application. Each boolean variable x and its negation ¬x become two nodes; each clause (a OR b) becomes two implications, ¬a -> b and ¬b -> a, as directed edges. Running SCC on this implication graph and checking whether any variable and its negation ended up in the same component tells you, in linear time, whether the whole formula is satisfiable — and if so, the condensation's topological order directly hands you a satisfying assignment.
