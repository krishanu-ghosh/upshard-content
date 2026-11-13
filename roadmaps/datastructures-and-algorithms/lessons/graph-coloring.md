---
title: "Graph Coloring"
description: Assigning colors to vertices so no edge joins two same-colored ones, why greedy coloring is fast but not optimal, and its use in compiler register allocation.
roadmap: datastructures-and-algorithms
node: graph-coloring
group: more-graph-algorithms
video:
  provider: youtube
  id: "3DcG72v2_n0"
  start: 0
  end: null
creator:
  name: TutorialsPoint
  channel_url: https://www.youtube.com/@TutorialsPoint_
duration_seconds: 565
level: intermediate
prerequisites: ["graph-representation"]
takeaways:
  - "Graph coloring assigns a color (or label) to every vertex such that no two vertices connected by an edge share the same color — the smallest number of colors needed for a given graph is its chromatic number."
  - "Greedy coloring processes vertices in some fixed order and assigns each one the lowest-numbered color not already used by any of its already-colored neighbors, which always produces a valid coloring but not necessarily the minimum one."
  - "Greedy coloring never uses more than (max degree + 1) colors, since any vertex has at most that many neighbors and so can never run out of colors in that range — but this bound can be far above the true chromatic number."
  - "The vertex order fed into greedy coloring changes the result: a deliberately bad order on a graph that's actually 2-colorable (like a 'crown' or bipartite-like structure) can force the greedy algorithm into using many more colors than necessary."
  - "Register allocation in compilers models each live variable as a vertex, draws an edge between two variables whenever their live ranges overlap (an 'interference graph'), and colors that graph with k colors, one per physical register — a coloring failure means some variable must be spilled to memory."
check_questions:
  - q: "A graph is bipartite (2-colorable), but greedy coloring assigns it 4 colors under a particular vertex order. Did greedy coloring make a mistake?"
    a: "No — greedy coloring is only guaranteed to produce *a* valid coloring, never promised to find the minimum one. A poor vertex order can cause a vertex to have neighbors of several different already-assigned colors even though a smarter order would have colored them more efficiently, inflating the color count well above the graph's actual chromatic number of 2."
  - q: "Why is (max degree + 1) colors always enough for greedy coloring, regardless of vertex order?"
    a: "When greedy coloring reaches any vertex v, at most deg(v) of its neighbors have already been colored (since v has exactly deg(v) neighbors total), so at most deg(v) colors are 'forbidden' for v. Since deg(v) is at most the graph's maximum degree, there is always at least one free color among the first (max degree + 1) colors, so greedy coloring can never get stuck."
  - q: "Why does a compiler model register allocation as graph coloring instead of just assigning registers in the order variables are declared?"
    a: "Two variables can share a physical register only if their live ranges (the span of code where each one's value is still needed) never overlap — assigning naively by declaration order ignores this and risks two live variables overwriting each other's register. Building an interference graph (edge = overlapping live ranges) and coloring it with k colors guarantees that any two variables sharing a color truly never conflict, and a graph that can't be colored with the available k registers tells the compiler exactly when it must spill a variable to memory instead."
deeper:
  - label: "Graph Coloring Problem — GeeksforGeeks"
    url: "https://www.geeksforgeeks.org/dsa/graph-coloring-applications/"
related_formulas: []
---

## The problem and why it's hard

**Graph coloring** asks: assign each vertex a color from some palette such that no edge connects two vertices of the same color. The minimum number of colors that makes this possible for a given graph is its **chromatic number**, written χ(G). Some graphs are easy — a tree is always 2-colorable, since you can alternate colors as you walk away from any root and never create a same-color edge (trees have no cycles to force a conflict). A cycle with an odd number of vertices, though, needs 3 colors no matter how you arrange them, because alternating two colors around an odd cycle always leaves one adjacent pair matching at the point where it wraps around.

Determining χ(G) exactly is NP-hard in general — there's no known efficient algorithm that always finds the true minimum, and even checking whether a graph is 3-colorable is NP-complete. This is what makes **greedy coloring** valuable: it's not optimal, but it's fast and always produces *some* valid coloring, which is often good enough in practice.

## Greedy coloring, mechanically

Fix any order of the vertices — say, v1, v2, v3, .... Process them in that order, and for each vertex, look at the colors already assigned to its neighbors (only the ones processed so far have colors), and assign the **smallest-numbered color not among them**. That's the entire algorithm. It never backtracks and never reconsiders a vertex once colored.

This always produces a valid coloring because, by the time a vertex is colored, any neighbor sharing an edge with it either already has a different color locked in (so the algorithm deliberately avoids repeating it) or doesn't have a color yet (so it can't conflict — when *that* neighbor is processed later, it will look at this vertex's already-assigned color and avoid it in turn).

**The degree bound:** greedy coloring never needs more than Δ+1 colors, where Δ is the graph's maximum degree. When the algorithm reaches any vertex v, at most deg(v) of its neighbors have been colored already (it has no more neighbors than that), so at most deg(v) colors are forbidden — and since deg(v) ≤ Δ, there's always at least one free color among the first Δ+1.

## Why greedy coloring isn't optimal

The gap between greedy's result and the true chromatic number depends entirely on vertex order. Consider a "crown" graph: two groups of vertices, A = {a1, a2} and B = {b1, b2}, where every vertex in A connects to every vertex in B except its "partner" (a1–b2, a1–b3 connect, but not a1–b1, and so on, generalized to larger groups). Such graphs are often 2-colorable (A gets color 1, B gets color 2), since all the edges run between the groups. But if you order the vertices by interleaving them adversarially — a1, b1, a2, b2, ... — the greedy algorithm can be forced to use many more colors, because at each step a vertex's already-colored neighbors happen to span every color used so far. The algorithm isn't broken; it's just making locally optimal choices that compound into a globally poor result. Smarter orderings — like coloring the highest-degree vertices first (a heuristic called **largest-degree-first** or, more refined, **DSATUR**, which always colors the vertex with the fewest remaining valid colors next) — tend to produce far better results in practice, though neither guarantees optimality either.

## Register allocation: a real use

Compilers use graph coloring to assign a program's variables to a small, fixed number of physical CPU registers. First, the compiler computes each variable's **live range** — the span of instructions during which its current value might still be read. Then it builds an **interference graph**: one vertex per variable, with an edge between two variables whenever their live ranges overlap (meaning they can't safely share a register, since one would overwrite the other mid-use). Coloring this graph with k colors, where k is the number of available physical registers, gives a valid register assignment — any two variables that interfere are guaranteed different registers. If the graph can't be colored with k colors, the compiler picks a variable to **spill**: store it in memory instead of a register for part of its lifetime, which shortens its live range (and the resulting edges) enough to make coloring possible. This exact idea — model conflicts as a graph, color it, spill on failure — is also how exam/meeting schedulers resolve "these two things can't happen at the same time slot" constraints, treating each time slot as a color.
