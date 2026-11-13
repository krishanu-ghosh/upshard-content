---
title: "Divide & Conquer and the Master Theorem"
description: Generalizing divide/conquer beyond mergesort and binary search, and using the Master Theorem to read off a recurrence's runtime by inspection.
roadmap: datastructures-and-algorithms
node: divide-and-conquer-and-the-master-theorem
group: algorithm-design-techniques
video:
  provider: youtube
  id: "OynWkEj0S-s"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 1010
level: intermediate
prerequisites: [searching-algorithms-and-recursion]
takeaways:
  - "Divide and conquer is a three-step pattern — split the problem into a smaller subproblems of size n/b, solve each recursively, then combine the subresults — and mergesort and binary search are just two instances of it, not the whole technique."
  - "Every divide-and-conquer algorithm's cost is captured by a recurrence T(n) = aT(n/b) + f(n), where a is the branching factor, n/b is the subproblem size, and f(n) is the cost of dividing and combining outside the recursive calls."
  - "The Master Theorem solves that recurrence by comparing f(n) against n^(log_b a), the cost implied by the recursion tree's branching alone, without needing to expand the recursion tree by hand."
  - "Which of the three Master Theorem cases applies tells you where the recurrence's cost is concentrated: at the leaves (recursion dominates), spread evenly across all levels (balanced), or at the root (the combine step dominates)."
  - "The Master Theorem only applies to recurrences of this exact shape with constant a and b; recurrences with a or b that depend on n, or a decreasing-by-subtraction form like T(n-1), need different tools entirely."
check_questions:
  - q: "In T(n) = aT(n/b) + f(n), what do a, b, and f(n) each represent in terms of the actual algorithm, and why does the Master Theorem compare f(n) to n^(log_b a) instead of just comparing it to n or n log n directly?"
    a: "a is how many subproblems each call spawns, b is the factor each subproblem shrinks by, and f(n) is the non-recursive work (splitting and combining) at that call. n^(log_b a) is the amount of work the branching alone would produce if the recursion tree's leaves did O(1) work each — it's the 'baseline' cost coming purely from the recursion's shape. Comparing f(n) to that baseline tells you whether the extra per-level work added by f(n) is small enough to be dominated by the recursion (case 1), matched to it (case 2), or large enough to dominate the recursion itself (case 3) — that comparison is what the three cases are built around, so it has to be the baseline, not an arbitrary reference like n or n log n."
  - q: "Mergesort has the recurrence T(n) = 2T(n/2) + O(n). Which Master Theorem case applies, and why does that case produce an extra log n factor over just n?"
    a: "Here a=2, b=2, so n^(log_b a) = n^(log_2 2) = n^1 = n, which matches f(n) = O(n) exactly — this is case 2 (f(n) = Theta(n^(log_b a))), where the work is balanced across all log n levels of the recursion tree rather than concentrated at the top or bottom. Case 2's result multiplies the per-level cost by an extra log n factor (giving O(n^(log_b a) log n) = O(n log n)) specifically because every one of the log n levels contributes the same Theta(n) amount of work, so the total is n work times log n levels, not just n."
  - q: "Why can't you apply the Master Theorem to a recurrence like T(n) = T(n-1) + O(1) (e.g. simple linear recursion), even though it also describes a recursive algorithm's cost?"
    a: "The Master Theorem is specifically built for recurrences of the form T(n) = aT(n/b) + f(n), where the subproblem shrinks by a multiplicative factor (dividing by b). T(n-1) shrinks the subproblem by subtracting a constant, not dividing — that's a fundamentally different recursion tree shape (depth n instead of depth log n), and the n^(log_b a) comparison the theorem is built around doesn't correspond to anything meaningful here. This kind of recurrence is solved directly (it unrolls to O(n)), not through the Master Theorem machinery."
deeper:
  - label: "Master Theorem — worked examples (Abdul Bari)"
    url: "https://www.youtube.com/watch?v=kGcO-nAm9Vc"
related_formulas: []
---

## Beyond mergesort and binary search

Divide and conquer is usually first taught through two examples — mergesort and binary search — and it's easy to walk away thinking the technique *is* those two algorithms. It isn't. The pattern is a three-step template: **divide** the problem into some number of smaller subproblems of the same kind, **conquer** each subproblem by recursing (until a small enough base case is solved directly), and **combine** the subresults into a solution for the original problem. Mergesort divides into two halves, conquers by recursively sorting each, and combines by merging two sorted lists in linear time. Binary search divides into one half (and discards the other), conquers by recursing, and combines trivially since there's nothing left to merge. But the same template also produces Karatsuba's O(n^1.585) integer multiplication, Strassen's O(n^2.807) matrix multiplication, the closest-pair-of-points algorithm, and quickselect's cousin quicksort. The combine step is where these algorithms actually differ from each other — it's often the most expensive or cleverest part, not an afterthought.

## The recurrence every divide-and-conquer algorithm produces

Once you commit to a divide-and-conquer design, its running time is described by a recurrence of the shape:

**T(n) = a·T(n/b) + f(n)**

Here `a` is the number of subproblems the divide step creates, `n/b` is each subproblem's size (the original shrinks by a factor of `b`), and `f(n)` is the cost of everything that *isn't* the recursive calls — splitting the input and combining the results at that level. Mergesort is T(n) = 2T(n/2) + O(n): two subproblems, each half the size, and O(n) work to merge. Binary search is T(n) = T(n/2) + O(1): one subproblem, half the size, constant work to decide which half to keep.

Solving this recurrence by hand — drawing the recursion tree, summing the work at every level — works, but it's slow and easy to get wrong. The Master Theorem turns that derivation into a lookup.

## The Master Theorem: comparing f(n) to the recursion's own appetite

The key quantity is **n^(log_b a)** — the total work the recursion's branching structure alone would imply if every leaf did O(1) work. The Master Theorem compares `f(n)` (the actual per-call overhead) against this baseline and falls into one of three cases:

**Case 1: f(n) = O(n^(log_b a - ε)) for some ε > 0.** f(n) grows strictly slower than the baseline. The recursion dominates: almost all the work happens at the leaves, and **T(n) = Θ(n^(log_b a))**. Example: T(n) = 8T(n/2) + O(n²). Here n^(log_2 8) = n³, which beats f(n) = n², so T(n) = Θ(n³).

**Case 2: f(n) = Θ(n^(log_b a) · log^k n)** for some k ≥ 0. f(n) matches the baseline, give or take a log factor. The work is spread evenly across all O(log n) levels of the tree, and **T(n) = Θ(n^(log_b a) · log^(k+1) n)**. Mergesort is the k=0 case: f(n) = Θ(n) matches n^(log_2 2) = n exactly, giving T(n) = Θ(n log n).

**Case 3: f(n) = Ω(n^(log_b a + ε))** for some ε > 0, plus a technical "regularity" condition (a·f(n/b) ≤ c·f(n) for some c < 1). f(n) dominates the recursion, so the combine step at the root is what matters, and **T(n) = Θ(f(n))**. Example: T(n) = 2T(n/2) + O(n²). f(n) = n² beats n^(log_2 2) = n, so T(n) = Θ(n²) — same as if there were no recursion at all.

## Why the gaps matter

Notice the cases don't cover every possible f(n) — there are gaps between "strictly polynomially smaller," "polynomially matching up to a log factor," and "strictly polynomially larger." A recurrence like T(n) = 2T(n/2) + n/log n falls in one of these gaps (f(n) is asymptotically between n and n log n in a way that isn't a clean polynomial separation), and the Master Theorem simply doesn't apply — you'd need the more general Akra-Bazzi method or a direct recursion-tree argument instead. Knowing the Master Theorem's boundary is as important as knowing its three cases: it's a fast tool for a specific, common recurrence shape, not a universal solver.
