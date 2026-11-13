---
title: "Branch and Bound"
description: Pruning a search tree with an optimality bound instead of a feasibility check, applied to 0/1 knapsack and the traveling salesman problem.
roadmap: datastructures-and-algorithms
node: branch-and-bound
group: algorithm-design-techniques
video:
  provider: youtube
  id: "yV1d-b_NeK8"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 648
level: intermediate
prerequisites: ["randomized-algorithms"]
takeaways:
  - "Backtracking prunes a search tree on feasibility: it abandons a partial solution the instant it violates a hard constraint (e.g. exceeds the knapsack's capacity), but it still explores every feasible partial solution in full."
  - "Branch and bound prunes on optimality: at every node it computes an upper bound (for maximization) on the best objective value any completion of that partial solution could possibly achieve, and abandons the node if that bound is no better than the best complete solution already found, even though the partial solution is still perfectly feasible."
  - "The bound must be 'admissible' — it has to be provably at least as good as any real completion could achieve (never underestimate the best case) — otherwise the algorithm risks pruning away the actual optimal solution."
  - "For 0/1 knapsack, a standard bound relaxes the 0/1 constraint to allow fractional items: solving the fractional (greedy, sort-by-value/weight) version of the remaining items at a node gives an upper bound on what any 0/1 completion from that node could achieve."
  - "The order nodes are explored in matters for how much pruning happens in practice: a best-first strategy (always expand the node with the most promising bound next) tends to find a strong complete solution early, which immediately tightens the pruning threshold for every other node, whereas a naive depth-first order may waste time in weak branches before stumbling onto a good solution."
check_questions:
  - q: "Backtracking and branch and bound both build the same search tree of partial solutions for 0/1 knapsack. What specifically does branch and bound check at each node that backtracking does not, and why does that extra check let it prune nodes backtracking would still explore?"
    a: "Backtracking only checks feasibility — does this partial solution still fit in the capacity? If yes, it's explored further regardless of how good the final answer from this branch could possibly be. Branch and bound additionally computes an upper bound on the best possible value any completion of this node could reach (e.g. via the fractional relaxation) and compares it against the best complete, feasible solution found anywhere so far. If the bound is no better than that incumbent, the entire subtree is abandoned even though every solution in it is perfectly feasible — backtracking has no way to make that comparison because it never computes a 'best possible from here' estimate, only a yes/no feasibility check."
  - q: "Why does the knapsack bound use the fractional relaxation (allowing partial items) rather than just summing the full values of all remaining items regardless of weight?"
    a: "Summing the full value of every remaining item ignores the capacity constraint entirely and would almost always overestimate wildly, producing a bound so loose it prunes almost nothing — it would rarely be beaten by the actual incumbent, defeating the purpose. The fractional relaxation is tighter because it does account for capacity: it greedily fills the remaining capacity with the best value-per-weight items, allowing the last item to be split fractionally. That's still an overestimate relative to the true 0/1 problem (since real items can't be split), which is exactly what admissibility requires — it never underestimates — but it's a much closer, more useful overestimate, so it prunes far more aggressively while still never cutting off the true optimum."
  - q: "If a branch-and-bound implementation used a bound that could sometimes be lower than what the best actual completion of that node would achieve, what would go wrong, concretely?"
    a: "The algorithm could prune a subtree whose true best achievable value is actually higher than the current incumbent, just because the (broken) bound happened to understate it — the real optimal solution could be sitting inside that discarded subtree. The algorithm would still terminate and return *some* feasible solution, but it could silently return a suboptimal one with no indication anything went wrong, which is worse than backtracking's guaranteed-correct exhaustive search; the entire value of branch and bound (pruning safely) depends on the bound never underestimating what's achievable from that node (for a maximization problem), i.e. the bound must be admissible."
deeper:
  - label: "0/1 Knapsack & TSP with Branch & Bound — LC Search step by step"
    url: "https://www.youtube.com/watch?v=6yrSpEueDa4"
related_formulas: []
---

## Two different reasons to cut a branch

Both backtracking and branch and bound build the same conceptual object: a search tree where each node is a partial solution, extended one decision at a time until it either becomes a complete solution or gets abandoned. They differ entirely in *why* a node gets abandoned.

**Backtracking prunes on feasibility.** At each node it asks: does this partial solution still satisfy the problem's hard constraints? For 0/1 knapsack, that's "does the total weight so far still fit under the capacity?" The moment a constraint is violated, the branch is dead — correctly so, since no completion of an infeasible partial solution can ever become feasible. But backtracking has no concept of "good" versus "bad" among the feasible branches; it explores every feasible partial solution to completion (or until infeasibility), regardless of whether that branch has any realistic chance of beating a solution you've already found.

**Branch and bound prunes on optimality.** At each node, in addition to checking feasibility, it computes a **bound**: a number that is provably at least as good as the best possible value any completion of this specific partial solution could achieve. It compares that bound against the best complete solution found anywhere in the search so far (the "incumbent"). If the bound is no better than the incumbent, the entire subtree is discarded. Not because it's infeasible — because it's provably incapable of improving on what's already in hand. That's the structural addition over plain backtracking: a second pruning criterion based on optimism about the best case, layered on top of the feasibility check.

## Worked example: 0/1 knapsack

Items have weights and values; capacity is W; each item is either fully included or fully excluded (no fractions). The search tree branches on each item in turn: left child "include item i," right child "exclude item i."

At a node where items 1..k have been decided (included or excluded), with remaining capacity `w_rem` and accumulated value `v_so_far`, the bound is computed via the **fractional relaxation**: sort the remaining undecided items by value-per-weight ratio, greedily add them whole until one doesn't fit, then add as much of that last item as fits *fractionally*. The resulting value — call it B — is provably at least as large as any value achievable by any 0/1 (non-fractional) completion from this node, because allowing fractions can only ever do as well or better than requiring whole items. If `v_so_far + B` is no better than the best complete 0/1 solution found so far, this node and everything beneath it is pruned — even though plenty of feasible 0/1 completions might still exist down that branch.

This bound is **admissible**: it never underestimates what's achievable, it only ever overestimates (because the true 0/1 problem is more constrained than its fractional relaxation). That property is non-negotiable — an admissible bound guarantees the true optimum never gets pruned by mistake, since any subtree actually containing the optimum will always show a bound at least as high as that optimum's value, and therefore will never look "no better than the incumbent" until the incumbent reaches that same optimal value.

## Worked example: traveling salesman

The same structure extends to TSP's search tree, where a node is a partial tour (a prefix of cities visited in order) and branching decides which unvisited city to visit next. A common bound sums, for each city not yet finalized in the tour, the cost of its two cheapest incident edges, averages and combines them with the edges already fixed in the partial tour — giving a lower bound (TSP is a minimization problem, so bounds here are lower bounds on remaining cost) on the length of any completion. A partial tour whose bound already exceeds the best complete tour found so far is abandoned, exactly as in knapsack, just with the comparison direction flipped because TSP minimizes rather than maximizes.

## Why exploration order matters in practice

Branch and bound's pruning power depends entirely on how tight the incumbent is at the time a node is evaluated. A weak incumbent prunes almost nothing. That makes the *order* of exploration a real performance lever, even though it never changes correctness.

A **best-first** strategy always expands whichever open node currently has the most promising bound. It tends to drive toward a strong complete solution early, which immediately sharpens the pruning threshold for every other open node in the tree. A naive depth-first traversal might instead sink time into a mediocre branch before ever finding a solution worth comparing against, pruning far less along the way.

Both strategies explore the same conceptual tree and both are guaranteed correct. Best-first is simply the version that tends to make branch and bound's central trick — give up early because you can prove you can't win — kick in sooner.
