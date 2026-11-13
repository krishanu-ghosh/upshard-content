---
title: "Meet in the middle"
description: Splitting a 2^n search space into two independent 2^(n/2) halves and combining the results turns an exponential brute force into something fast for n up to ~40.
roadmap: datastructures-and-algorithms
node: meet-in-the-middle
group: problem-solving-patterns
video:
  provider: youtube
  id: "18sJ3mK173s"
  start: 0
  end: null
creator:
  name: Errichto Hard Algorithms
  channel_url: https://www.youtube.com/@Errichto2
duration_seconds: 4820
level: intermediate
prerequisites: [binary-search-on-the-answer]
takeaways:
  - "Meet in the middle splits a problem of size n into two independent halves of size n/2, enumerates all 2^(n/2) possibilities for each half separately, and then combines the two result sets — turning an O(2^n) brute force into roughly O(2^(n/2) log(2^(n/2)))."
  - "The technique is what makes subset-sum-style problems with n up to about 40 tractable: 2^40 is far too large to enumerate directly, but 2^20 (about a million) enumerated twice, each sorted and searched, is easily fast enough."
  - "Combining the two halves is usually a sort-plus-binary-search or a two-pointer merge: for subset sum, sort one half's sums and for each sum in the other half, binary-search for the complement that hits the target."
  - "The two halves must be genuinely independent — solvable without knowing anything about the other half — which is why the pattern applies to 'choose a subset of these n items' problems but not to problems where every choice depends on the full running state so far."
  - "Meet in the middle trades the exponential factor in the exponent for a square root: instead of O(2^n) you get roughly O(2^(n/2)), which is an enormous practical difference (2^40 vs. roughly 2^20) even though both are technically exponential in n."
check_questions:
  - q: "Why does splitting a subset-sum problem with n=40 items into two halves of 20 items each make it tractable when brute force on the full 40 is not?"
    a: "Brute force enumerates all 2^40 subsets directly (over a trillion), which is too slow. Meet in the middle enumerates 2^20 subsets of each half separately (about a million each, easily fast), then combines the two lists with a sort and binary search in roughly O(2^20 log(2^20)) — the exponential blowup is cut from the full n down to n/2 in the exponent."
  - q: "Why is sorting one half's results and binary-searching the other half's complement faster than comparing every pair of partial results directly?"
    a: "Comparing every pair of the two halves' results directly costs O(2^(n/2) * 2^(n/2)) = O(2^n), which throws away the whole benefit of splitting. Sorting one half (O(2^(n/2) log(2^(n/2)))) and binary-searching it for each element of the other half (O(2^(n/2) log(2^(n/2))) total) avoids ever comparing all pairs explicitly."
  - q: "Why can't meet in the middle be applied to a problem where each decision depends on the cumulative effect of all previous decisions, like a path-counting DP over a graph?"
    a: "Meet in the middle requires the two halves to be solvable independently and combined only at the end; if every decision depends on the running state built up by every prior decision, you can't enumerate one half without knowing what the other half already did, so there's no clean split point for the technique to exploit."
deeper:
  - label: "Meet in the Middle — USACO Guide"
    url: "https://usaco.guide/gold/meet-in-the-middle"
related_formulas: []
---

## The problem with brute-force subset enumeration

Many classic problems ask you to consider every subset of a set of n items — does
some subset sum to a target value, what's the closest achievable sum to a target,
how many subsets satisfy some property. The direct brute force enumerates all 2^n
subsets. For n=20 that's about a million, entirely fine. For n=40, 2^40 is over a
trillion — far too slow, even though 40 items still sounds like a small, easy input.
Meet in the middle is the technique that makes exactly this range of n (roughly
30-45) tractable without needing a fundamentally different algorithm.

## The mechanism: split, enumerate, combine

Split the n items into two halves, A and B, each of size n/2. Enumerate *all* 2^(n/2)
possible subset-sums of A, and separately all 2^(n/2) possible subset-sums of B —
each half is small enough to brute-force on its own. Now the question "does some
subset of the full n items sum to target T?" becomes "does some sum from A's list
plus some sum from B's list equal T?" — which you answer by sorting one list (say B's
sums) and, for every sum `a` in A's list, binary-searching B's sorted list for the
complement `T - a`. Enumerating each half costs O(2^(n/2)), sorting costs
O(2^(n/2) log(2^(n/2))), and the binary-search combination costs the same order — so
the total is roughly O(2^(n/2) * n/2), astronomically smaller than O(2^n * n) for
n=40.

## Worked example

Take items `[3, 7, 1, 8, 2, 5]` (n=6, small enough to trace by hand) and target
T=14. Split into A = `[3, 7, 1]` and B = `[8, 2, 5]`. Enumerate all 2^3=8 subset sums
of A: `{0, 3, 7, 1, 10, 4, 8, 11}` (empty set through full set). Enumerate all 2^3=8
subset sums of B: `{0, 8, 2, 5, 10, 13, 7, 15}`. Sort B's sums:
`[0, 2, 5, 7, 8, 10, 13, 15]`. Now for each sum `a` in A's list, binary-search for
`14 - a` in B's sorted list: for `a=7`, need `7` in B — present (subset `{2,5}` of B,
or `{7}` alone). So subset `{7}` from A plus `{2,5}` from B sums to 14. Found it in
roughly 8 + 8 + (8 log 8) operations, instead of checking all 2^6=64 subsets directly
— the advantage compounds rapidly as n grows, which is exactly why this matters most
for n in the 30-45 range where 2^n is intractable but 2^(n/2) is easy.

## Why the halves must be truly independent

The technique only works because a subset of A and a subset of B can be chosen with
no interaction between the choices — any subset of A combines validly with any subset
of B. This is why meet in the middle fits subset-selection problems (subset sum, closest
subset sum, counting subsets with a given XOR) but doesn't fit problems where later
choices depend on the specific path taken so far, like shortest-path or sequential DP
over a graph — there, the two "halves" can't be solved in isolation because what's
feasible in the second half depends on exactly which state the first half ended in,
not just on a combinable summary value.

## From binary search on values to binary search on a combined list

The previous lesson used binary search to narrow a range of candidate numeric
answers against a monotonic feasibility check. Here, binary search reappears in the
combination step — searching one sorted half's results for the complement needed to
hit a target — which is why that lesson is the natural prerequisite: meet in the
middle doesn't introduce a new searching primitive, it reuses binary search (or a
two-pointer merge) as the glue between two independently brute-forced halves.
