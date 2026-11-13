---
title: "Dynamic Programming & NP-Hard Problems"
description: Solving overlapping subproblems exactly once with memoization and tabulation, applied to knapsack and the traveling salesman problem.
roadmap: datastructures-and-algorithms
node: dynamic-programming-and-np-hard-problems
group: algorithm-design-paradigms
video:
  provider: youtube
  id: "nLmhmB6NzcM"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 1704
level: advanced
prerequisites:
  - backtracking
takeaways:
  - Dynamic programming applies when a problem has overlapping subproblems (the same smaller problem gets solved repeatedly) and optimal substructure (an optimal solution is built from optimal solutions to subproblems), and it exploits overlap by solving each distinct subproblem exactly once.
  - Memoization is top-down DP — write the natural recursive solution, then cache each subproblem's answer the first time it's computed so later calls with the same arguments are a lookup instead of a recomputation.
  - Tabulation is bottom-up DP — build a table of subproblem answers from the smallest up, so every larger answer is computed from already-known smaller ones with no recursion at all.
  - The 0/1 knapsack problem, which defeats greedy, is solved exactly by DP in pseudo-polynomial time by building a table indexed by item count and remaining capacity, where each cell answers "best value using the first i items with capacity w".
  - The traveling salesman problem and the longest-path problem are both NP-hard, and the Held-Karp DP algorithm solves TSP exactly in O(n² · 2ⁿ) time — astronomically better than the O(n!) of trying every route, but still exponential, which is the ceiling DP can't break for an NP-hard problem.
check_questions:
  - q: What two properties does a problem need before dynamic programming is the right tool, and how do they differ from what greedy needs?
    a: It needs overlapping subproblems and optimal substructure; it shares optimal substructure with greedy but, unlike greedy, doesn't require the greedy-choice property — DP tries all the relevant choices at each step and keeps the best, rather than committing to one choice irreversibly.
  - q: What's the practical difference between memoization and tabulation, given that both compute the same answers?
    a: Memoization is the recursive solution with a cache added, computing subproblems on demand top-down, while tabulation fills a table from the smallest subproblems upward with no recursion, which avoids call-stack overhead and is often faster in practice.
  - q: Why does an exact, optimal DP solution to the traveling salesman problem not make TSP a polynomial-time problem?
    a: Because the Held-Karp algorithm's O(n² · 2ⁿ) running time is still exponential in n — enormously better than trying every one of the (n-1)! possible routes, but not polynomial, which is exactly what makes TSP NP-hard rather than merely hard to brute-force.
deeper:
  - label: "Traveling Salesman Problem Dynamic Programming Held-Karp"
    url: "https://www.youtube.com/watch?v=-JjA4BLQyqE"
related_formulas: []
---

## Solve it once, remember it forever

Backtracking's weakness, when it has one, is doing the same work twice. Picture a recursive algorithm that, at each step, branches into a few smaller subproblems — and imagine two different branches each eventually asking the identical question: "what's the best way to solve this exact smaller case?" A plain recursive or backtracking approach answers that question freshly every single time it comes up, even though the answer never changes. When the same subproblem recurs exponentially many times across a recursion tree, that redundancy is where all the wasted time goes. **Dynamic programming (DP)** is the fix: identify that overlap, solve each distinct subproblem exactly once, and store the answer so every later occurrence is a cheap lookup instead of a full recomputation.

DP applies when a problem has two properties. **Optimal substructure** — shared with greedy — means an optimal solution to the whole problem is built from optimal solutions to its subproblems. **Overlapping subproblems** — the property greedy doesn't need and backtracking doesn't exploit — means those subproblems recur rather than being distinct every time. Where greedy commits to one choice per step and never looks back, DP effectively tries every relevant choice at each step but reuses subproblem answers so trying everything doesn't mean redoing everything.

## Two ways to organize the same idea

**Memoization** is the top-down version: write the natural recursive function exactly as the problem suggests, then add a cache (usually a hash map or array) keyed by the subproblem's arguments. Before computing, check the cache; after computing, store the result. The recursion structure stays intact — you're just making sure no distinct subproblem is ever solved twice.

**Tabulation** is the bottom-up version: instead of recursing downward from the big problem, start from the smallest subproblems (the base cases) and fill a table outward, so that by the time you need a larger subproblem's answer, every smaller answer it depends on is already sitting in the table. There's no recursion, no call stack, just iteration over the table in an order that respects dependencies. The two approaches compute identical answers; tabulation is often a little faster in practice (no function-call overhead) while memoization can be easier to write correctly, since it mirrors the problem's natural recursive definition.

## Worked example: 0/1 knapsack, exactly

The previous lesson showed that greedy gives the wrong answer on 0/1 knapsack — capacity 50, items (10, 60), (20, 100), (30, 120), greedy finds value 160 when the true optimum is 220. DP finds that optimum by building a table where cell `dp[i][w]` means: "the best achievable value using only the first *i* items, with total weight at most *w*." The recurrence for each cell is a straightforward choice: either skip item *i* (`dp[i-1][w]`), or, if it fits, take it (`value[i] + dp[i-1][w - weight[i]]`) — and `dp[i][w]` is the better of the two. Filling this table row by row (one row per item, one column per capacity from 0 to 50) takes O(n · W) time, where n is the item count and W is the capacity — for n = 3 and W = 50, that's 150 cells, each a constant-time comparison. The final cell, `dp[3][50]`, holds 220: the exact optimum, because every cell considers *both* options rather than greedily locking in one. This running time is called **pseudo-polynomial**: polynomial in the numeric value of W, but W itself can be exponentially large relative to the number of bits needed to write it down, so knapsack remains NP-hard in the general sense even though this DP solves any particular instance efficiently when W is reasonably sized.

## When DP meets a problem that stays hard

The **traveling salesman problem (TSP)** — find the shortest route that visits every city exactly once and returns to the start — looks like a natural fit for DP, and it is, but it illustrates DP's limits as clearly as its power. Brute force tries all (n-1)! possible orderings of the remaining cities, which is unusable past roughly a dozen cities. The **Held-Karp algorithm** applies DP instead: `dp[S][i]` means "the shortest path that visits exactly the set of cities S, ending at city i." Each state only needs to know *which* cities have been visited (as a bitmask) and *where* the path currently ends — not the order visited — and transitions by extending a path ending at some city in S to a new, unvisited city. This collapses the "same subproblem recurs" redundancy in essentially the same way as knapsack: many different visiting *orders* land on the same `(S, i)` state, and Held-Karp computes each such state once. The result is O(n² · 2ⁿ) time — for n = 20, that's roughly 20² × 2²⁰, around 400 million operations, versus 19! (about 10¹⁷) for brute force. That's a staggering improvement, but 2ⁿ is still exponential: TSP is NP-hard, and no known algorithm — DP included — solves it in time polynomial in n. The **longest-path problem** in a general graph has the same character: DP (via similar bitmask techniques, or relaxation-based approaches for DAGs specifically) can beat brute force enormously, but can't make an NP-hard problem polynomial. Dynamic programming, in other words, is exact and often dramatically faster than brute force — it just can't manufacture a shortcut around an NP-hardness bound that complexity theory says, as far as anyone has proven, doesn't exist.
