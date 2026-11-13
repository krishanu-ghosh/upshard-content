---
title: "Floyd-Warshall & All-Pairs Shortest Paths"
description: The O(V^3) DP recurrence that computes shortest paths between every pair of vertices at once, by considering each vertex in turn as a possible waypoint.
roadmap: datastructures-and-algorithms
node: floyd-warshall-all-pairs-shortest-paths
group: advanced-graph-algorithms
video:
  provider: youtube
  id: "4NQ3HnhyNfQ"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 953
level: intermediate
prerequisites: ["bellman-ford-and-negative-weights"]
takeaways:
  - "Floyd-Warshall solves the all-pairs shortest path problem directly: instead of running a single-source algorithm once per vertex, it fills one V-by-V distance matrix that holds every pair's shortest distance simultaneously."
  - "Its recurrence asks one question per step: for an intermediate vertex k, is the path from i to j through k (dist[i][k] + dist[k][j]) shorter than the best path from i to j found using only intermediate vertices considered so far?"
  - "Looping k from 1 to V, and updating the whole matrix in place for each k, correctly builds up shortest paths that are allowed to route through progressively larger sets of intermediate vertices, until all V vertices are allowed and every entry is a true shortest-path distance."
  - "The algorithm costs O(V^3) time and O(V^2) space with three nested loops and no data structures beyond the matrix itself, which is simpler to implement correctly than V runs of Dijkstra or Bellman-Ford despite sometimes doing more total work."
  - "It beats running Dijkstra or Bellman-Ford from every source whenever V is small-to-moderate and the graph is dense (E close to V^2) or has negative edges without a negative cycle, since V times Dijkstra costs O(V*E*log V) and V times Bellman-Ford costs O(V^2*E) — both of which can exceed O(V^3) on dense graphs."
  - "A negative cycle shows up directly in the finished matrix as a negative value on the diagonal, dist[i][i] < 0, meaning there's a path from i back to itself that costs less than doing nothing."
check_questions:
  - q: "What does the intermediate vertex k actually represent in the Floyd-Warshall recurrence, and why does looping k from 1 to V (rather than picking one k) make the algorithm correct?"
    a: "k represents a vertex the path from i to j is allowed to pass through. After the loop has processed k = 1..m, dist[i][j] holds the shortest path from i to j that only routes through vertices in {1..m} as intermediate stops. Looping k through every vertex incrementally allows more and more vertices as potential waypoints, so by k = V every possible intermediate vertex has been considered and dist[i][j] is the true shortest path."
  - q: "On a dense graph with 500 vertices, why might Floyd-Warshall's O(V^3) actually beat running Dijkstra 500 times even though V^3 looks like a lot of work?"
    a: "Dense means E is close to V^2, so running Dijkstra once from every vertex costs roughly O(V * E log V) ≈ O(V^3 log V) — strictly worse than Floyd-Warshall's O(V^3), and that's before accounting for the overhead of maintaining a priority queue per run versus Floyd-Warshall's three plain nested loops over a flat array."
  - q: "Why is a negative value on the diagonal of the finished distance matrix, dist[i][i] < 0, specifically a sign of a negative cycle rather than just a negative edge somewhere?"
    a: "dist[i][i] starts at 0 (the trivial zero-length path from i to itself) and can only become negative if some path leaves i and returns to i with total weight less than zero — which is, by definition, a negative-weight cycle through i. A single negative edge that isn't part of a cycle back to i can't make dist[i][i] negative on its own."
deeper:
  - label: "Floyd-Warshall — All Pairs Shortest Paths — CP-Algorithms"
    url: "https://cp-algorithms.com/graph/all-pair-shortest-path-floyd-warshall.html"
related_formulas: []
---

## One matrix for every pair at once

Bellman-Ford and Dijkstra both answer a single-source question: shortest distances from one starting vertex to everywhere else. The natural way to get **all-pairs** shortest paths — every vertex to every other vertex — is to just run one of those algorithms once per vertex as the source. Floyd-Warshall instead attacks the all-pairs problem directly, building a single `V x V` matrix `dist[i][j]` where every cell gets filled in together, using a dynamic-programming recurrence over which vertices are allowed as waypoints.

## The recurrence: one vertex at a time as a waypoint

Initialize `dist[i][j]` to the direct edge weight from i to j (or infinity if no edge exists, 0 when i = j). Then, for each vertex k from 1 to V, update every cell:

```
dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```

The key idea is what `dist[i][j]` *means* at each stage: after processing k = 1 through m, `dist[i][j]` holds the shortest path from i to j that is only allowed to use vertices from `{1, ..., m}` as intermediate stops (i and j themselves are always allowed as endpoints). When vertex k is being processed, the update asks exactly one question for every pair (i, j): does routing through k — going i to k, then k to j, using only previously-allowed intermediates for each half — beat the best path found without k available as a waypoint? Since both halves of that candidate path (i-to-k and k-to-j) were already correctly computed using only vertices `{1, ..., k-1}`, extending the allowed set to include k and taking the minimum is enough to correctly fold k in. Updating in place (reusing the same matrix rather than keeping a separate matrix per k) is safe because `dist[i][k]` and `dist[k][j]` don't change during the update for k's own iteration — a path through k can't usefully route through k again.

## Worked example

Take three vertices {1, 2, 3} with edges 1→2 weight 4, 2→3 weight -2, and a direct edge 1→3 weight 10. The matrix initializes with `dist[1][3] = 10` (the direct edge), `dist[1][2] = 4`, `dist[2][3] = -2`, and `dist[1][1] = dist[2][2] = dist[3][3] = 0`. When k = 1 is processed, no pair improves by routing through vertex 1 (nothing points into it here). When k = 2 is processed, every pair checks whether detouring through vertex 2 helps — in particular, `dist[1][3]` is compared against `dist[1][2] + dist[2][3] = 4 + (-2) = 2`. Since 2 < 10, `dist[1][3]` updates to 2, having discovered that routing through vertex 2 beats the direct edge. When k = 3 runs, nothing points out of vertex 3 in this graph, so no further improvement happens. The final matrix correctly reports the 1-to-3 shortest distance as 2, something no single lookup at the edge list could have shown directly. This is the entire algorithm, repeated for every pair and every candidate waypoint: check if detouring through the current vertex helps, and keep the best answer found so far.

## Cost and when it wins

Three nested loops (k, then i, then j) over V vertices gives `O(V^3)` time and `O(V^2)` space for the matrix — no priority queue, no edge list traversal logic, just array reads and a comparison. Running Dijkstra from every vertex costs `O(V * E log V)`; running Bellman-Ford from every vertex costs `O(V^2 * E)`. On a **dense** graph, where E approaches `V^2`, both of those become `O(V^3 log V)` and `O(V^4)` respectively — both worse than Floyd-Warshall's flat `O(V^3)`. Floyd-Warshall also handles negative edge weights natively (it just needs the absence of a negative cycle to produce meaningful finite answers), so it's a strong default whenever V is small enough that `V^3` is affordable (roughly a few hundred to low thousands of vertices) and either the graph is dense or negative weights are in play. Its real selling point beyond raw asymptotics is simplicity: three loops and an array are far easier to get right under time pressure than V correctly-instrumented runs of a priority-queue-based algorithm, and the full matrix is useful on its own whenever many-to-many queries (not just one source) are the actual goal.
