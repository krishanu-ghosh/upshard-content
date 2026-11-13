---
title: "A* Search"
description: Dijkstra plus a goal-directed heuristic — how f = g + h skips exploring every direction, and the admissibility/consistency conditions that keep it optimal.
roadmap: datastructures-and-algorithms
node: a-star-search
group: advanced-graph-algorithms
video:
  provider: youtube
  id: "ySN5Wnu88nE"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 844
level: intermediate
prerequisites: ["d-ary-heaps-and-indexed-priority-queues", "floyd-warshall-all-pairs-shortest-paths"]
takeaways:
  - "A* is Dijkstra with one change: instead of ordering the priority queue by g(n), the exact known cost from the start to n, it orders by f(n) = g(n) + h(n), where h(n) is a heuristic estimate of the remaining cost from n to the goal."
  - "Because h(n) biases the search toward the goal, A* expands far fewer nodes than Dijkstra on the same graph, which explores outward in all directions equally since it has no notion of where the goal is."
  - "A heuristic is admissible if it never overestimates the true remaining cost to the goal for any node — admissibility alone guarantees A* finds the optimal path, though it may revisit nodes."
  - "A heuristic is consistent (monotone) if, for every edge (n, n'), h(n) <= cost(n, n') + h(n') — consistency is stronger than admissibility, implies it, and additionally guarantees f-values never decrease along any path, which lets A* finalize each node the first time it's popped, exactly like Dijkstra."
  - "A* is the standard pathfinding algorithm in games, maps, and robotics because a cheap, problem-specific heuristic (straight-line or Manhattan distance) turns an otherwise uniform search into one that beelines toward the goal while still guaranteeing the shortest path."
check_questions:
  - q: "If you set h(n) = 0 for every node, what algorithm does A* become, and why?"
    a: "A* becomes plain Dijkstra, because f(n) = g(n) + 0 = g(n), so the priority queue is ordered purely by the known cost so far with no goal-directed bias at all — h(n) = 0 is trivially admissible and consistent, just useless as a heuristic."
  - q: "A heuristic slightly overestimates the true remaining distance for some node. What specifically can go wrong, and why doesn't admissibility being 'mostly right' save it?"
    a: "An overestimating heuristic can make a node on the true shortest path look worse (higher f) than it actually is, causing A* to expand a different, actually-longer path to the goal first and return it as the answer before the true shortest path's nodes are ever reconsidered. Admissibility requires never overestimating for any node, because a single violation anywhere on the optimal path is enough to make A* miss it."
  - q: "Why does a consistent heuristic let A* finalize a node's distance the moment it's popped from the priority queue, the same way Dijkstra does?"
    a: "Consistency (h(n) <= cost(n,n') + h(n')) guarantees that f-values are non-decreasing along any path explored from the start, which means the first time a node is popped with the lowest f in the queue, no later-discovered path through an unexpanded node could possibly produce a smaller f for it — mirroring exactly why Dijkstra's pop-and-finalize step is safe with non-negative edge weights."
deeper:
  - label: "Introduction to A* — Red Blob Games"
    url: "https://www.redblobgames.com/pathfinding/a-star/introduction.html"
related_formulas: []
---

## Dijkstra, but pointed at the goal

Dijkstra's algorithm finds the shortest path to every vertex by always expanding the frontier node with the smallest known cost-so-far, `g(n)`. That makes it correct but blind: it has no idea where the goal is, so on a map it expands outward as a uniform, ever-growing circle centered on the start, wasting enormous effort exploring directions that lead away from the destination. A* fixes exactly this, and nothing else. It keeps Dijkstra's machinery — a priority queue, relaxation of neighbors, tracking the best known cost to reach each node — but changes the priority key from `g(n)` alone to:

```
f(n) = g(n) + h(n)
```

where `h(n)` is a **heuristic**: a problem-specific, cheaply computed *estimate* of the remaining cost from n to the goal. A node with a low `g(n)` but a huge estimated remaining distance gets deprioritized in favor of a node that's a bit further traveled but clearly closer to the goal. The algorithm's loop is otherwise identical to Dijkstra's: pop the lowest-`f` node, relax its neighbors' `g` values, push updates, repeat until the goal is popped.

## Two conditions that keep it honest

Adding a heuristic risks correctness — a bad estimate could make A* confidently walk right past the true shortest path. Two properties rule that out, at different strengths.

**Admissibility**: `h(n)` never overestimates the true remaining cost to the goal, for every node n. This alone is enough to guarantee A* returns the optimal path, because the true shortest path's f-values are always underestimated or exact, so a sub-optimal path can never look permanently cheaper than it really is — A* will eventually reconsider and correct course once the true path's low real cost becomes visible. The cost of admissibility alone is that A* may need to re-open nodes it already finalized, since f-values along a path aren't guaranteed to only increase.

**Consistency (monotonicity)**: a stricter condition, requiring that for every edge (n, n'), `h(n) <= cost(n, n') + h(n')` — essentially a triangle inequality on the heuristic itself. Consistency implies admissibility, and it buys something Dijkstra relies on: f-values never decrease along any explored path. That means the moment a node is popped with the smallest f in the queue, no cheaper route to it can possibly be found later — it can be finalized immediately, just like in Dijkstra, with no re-opening needed. Virtually every practical A* heuristic used in pathfinding (straight-line "as the crow flies" distance, Manhattan distance on a grid with uniform step costs) is both admissible and consistent, which is why A* implementations typically finalize-on-pop without worrying about the weaker admissible-only case.

## Worked example: grid pathfinding

Consider a grid where movement costs 1 per step in any of 4 directions, start at (0,0), goal at (5,5), with no obstacles. Dijkstra explores every cell within Manhattan distance k from the start before touching any cell at distance k+1, regardless of direction — it would explore roughly a full diamond of ~61 cells before reaching the goal at distance 10. A* with `h(n)` = Manhattan distance from n to (5,5) immediately prioritizes cells like (1,1) over (1,-3)-direction detours, since their f-values (g + h) stay pinned near the true optimal cost of 10 the entire way. In an open grid, A* with this heuristic expands close to the minimal 11 cells along the direct path, because every off-path cell has a strictly higher f than the true shortest distance, so it's never competitive for the priority queue's top slot. This is why A* dominates pathfinding in games, GPS routing, and robot motion planning: the heuristic is nearly free to compute (a distance formula), it's provably safe to use when admissible/consistent, and it turns a search that would otherwise explore the entire reachable region into one that explores a narrow corridor toward the goal.
