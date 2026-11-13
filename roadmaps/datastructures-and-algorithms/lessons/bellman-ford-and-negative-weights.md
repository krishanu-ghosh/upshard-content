---
title: "Bellman-Ford & Negative Weights"
description: Why Dijkstra's greedy finalization breaks with negative edges, and how repeated edge relaxation finds shortest paths anyway — and detects negative cycles.
roadmap: datastructures-and-algorithms
node: bellman-ford-and-negative-weights
group: advanced-graph-algorithms
video:
  provider: youtube
  id: "lyw4FaxrwHg"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 900
level: intermediate
prerequisites: ["strongly-connected-components", "shortest-path-algorithms"]
takeaways:
  - "Dijkstra's correctness depends on finalizing each vertex's distance the moment it's popped from the priority queue, and that assumption only holds when every edge weight is non-negative — a later negative edge can still shrink a \"finalized\" distance, producing a silently wrong answer rather than a crash."
  - "Bellman-Ford relaxes every edge in the graph, in any order, and repeats this full pass |V|-1 times, where relaxing an edge (u,v) means setting dist[v] = min(dist[v], dist[u] + w(u,v))."
  - "|V|-1 passes are exactly enough because any shortest path (in a graph with no negative cycle) uses at most |V|-1 edges, and each full pass guarantees that all shortest paths using one more edge than before are now correctly computed."
  - "The total cost is O(V*E), since each of the |V|-1 passes relaxes all E edges — asymptotically worse than Dijkstra's O(E log V), which is the price paid for tolerating negative weights."
  - "Running one extra, |V|th pass after the algorithm 'finishes' detects a negative cycle: if any edge can still be relaxed, some vertex's distance would keep shrinking forever, which is only possible if a negative-weight cycle is reachable from the source."
check_questions:
  - q: "A graph has a single negative edge but no negative cycle. Why can Dijkstra still return a wrong shortest-path distance for some vertex, rather than just running slower?"
    a: "Dijkstra pops the vertex with the smallest tentative distance and treats it as final, never revisiting it. If a negative edge is discovered later that would have shortened the path to an already-finalized vertex, Dijkstra has no mechanism to go back and fix it — it simply reports the stale, too-large distance as correct."
  - q: "Why does Bellman-Ford need exactly |V|-1 rounds of relaxation, not more and not fewer, to guarantee correct shortest-path distances (assuming no negative cycle)?"
    a: "A shortest path in a graph with V vertices and no negative cycle visits each vertex at most once, so it has at most |V|-1 edges. Each full relaxation pass is guaranteed to correctly compute every shortest path that uses at most k edges after k passes, so after |V|-1 passes, every shortest path — regardless of its edge count up to the maximum possible — is correct."
  - q: "After |V|-1 passes, you run one more pass and an edge (u,v) still relaxes successfully. What does that actually prove, and why can't the algorithm just report a slightly-too-long distance instead?"
    a: "It proves v is reachable from a negative-weight cycle: if distances could still legitimately improve after every path up to length |V|-1 was already accounted for, the only remaining explanation is a cycle whose total weight is negative, which can be traversed infinitely to drive the \"shortest\" distance to negative infinity — so there is no finite correct answer to report."
deeper:
  - label: "Bellman-Ford Algorithm — CP-Algorithms"
    url: "https://cp-algorithms.com/graph/bellman_ford.html"
related_formulas: []
---

## Where Dijkstra's assumption breaks

Dijkstra's algorithm works by repeatedly picking the unvisited vertex with the smallest tentative distance and declaring that distance final. That greedy step is only safe because, with non-negative edge weights, nothing discovered later can ever make a path to an already-finalized vertex shorter — every alternative path has to go through some other vertex first, and adding more non-negative edges never decreases a total. A negative edge destroys that guarantee. Picture vertices A, B, C with edges A→B weight 2, A→C weight 5, and B→C weight -4. Dijkstra finalizes A at 0, then B at 2 (smallest remaining), and finalizes C at min(5, 2 + (-4)) = min(5, -2) = -2 only if it happens to relax C through B before finalizing C — but if C had already been popped and finalized at 5 before B was processed (plausible under certain weight distributions with more vertices in the mix), Dijkstra would report 5 and never revisit it. The bug isn't a crash; it's a confidently wrong number.

## Relax everything, repeatedly

Bellman-Ford sidesteps the finalize-once assumption entirely by never trusting any distance until the very end. The core operation, **relaxation**, is simple: for an edge (u, v) with weight w, if `dist[u] + w < dist[v]`, update `dist[v] = dist[u] + w`. The algorithm initializes `dist[source] = 0` and every other distance to infinity, then does something almost naively brute-force: relax **every edge in the graph**, in any fixed order, and repeat this full sweep `|V| - 1` times total.

Why does this converge? After the first pass, every distance that's reachable via a single edge from the source is correct. After the second pass, every distance reachable via a path of at most two edges is correct, because the second pass relaxes edges extending those one-edge paths by one more hop. By induction, after the k-th pass, every shortest path using at most k edges has been correctly computed — relaxation along the path's edges, in some order across the k passes, eventually captures it regardless of which order the edges were processed in during any individual pass. Since a shortest simple path in a graph with no negative cycle visits at most `|V| - 1` distinct intermediate vertices (any more and it would revisit a vertex, implying a cycle, which can be removed without increasing a non-negative-cycle path's length), `|V| - 1` passes are provably sufficient to capture every shortest path, however many edges it uses.

## The cost, and the bonus pass

Each pass touches every edge once, so `|V| - 1` passes cost `O(V * E)` total — markedly worse than Dijkstra's `O(E log V)` with a binary heap, which is exactly the tradeoff: Bellman-Ford gives up speed for tolerance of negative weights.

That tolerance has a limit, though — negative weights are fine, but a **negative cycle** (a cycle whose total edge weight sums to less than zero) breaks the very idea of "shortest path," since looping it indefinitely drives the path weight toward negative infinity. The fix is elegant: after the standard `|V| - 1` passes complete, run one more pass. If any edge still relaxes successfully — some `dist[v]` can still be shrunk — that's proof a negative cycle is reachable from the source, because every legitimate shortest path (one using at most `|V| - 1` edges) was already locked in by the previous passes. Any further improvement can only come from a path that keeps "cheating" by looping through a cycle that makes it cheaper each time around, and the algorithm reports a negative cycle rather than a number.
