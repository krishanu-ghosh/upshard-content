---
title: "Greedy Algorithms"
description: Making the locally best choice at every step, why that provably works for some problems, and why it quietly fails for others.
roadmap: datastructures-and-algorithms
node: greedy-algorithms
group: algorithm-design-paradigms
video:
  provider: youtube
  id: "ARvQcqJ_-NY"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 900
level: intermediate
prerequisites: []
takeaways:
  - "A greedy algorithm builds a solution by repeatedly making whichever choice looks best right now, never reconsidering an earlier choice once it's made."
  - "Greedy only gives a correct, globally optimal answer when the problem has the greedy-choice property (a locally optimal pick is always part of some globally optimal solution) and optimal substructure (an optimal solution to the whole problem contains optimal solutions to its subproblems)."
  - "Activity selection and Huffman coding are classic problems where greedy is not just fast but provably optimal, and both of the minimum-spanning-tree algorithms from the previous module (Prim's and Kruskal's) are greedy algorithms in disguise."
  - "The 0/1 knapsack problem is the standard counterexample: picking the item with the best value density first can lock in a choice that an exhaustive search would have avoided, so greedy gives a fast but wrong answer there."
  - "When a problem's subproblems overlap and a locally best choice can't be trusted, that's the signal to reach for dynamic programming instead, which is where the next two lessons go."
check_questions:
  - q: What two properties does a problem need for a greedy algorithm to be provably optimal, not just fast?
    a: It needs the greedy-choice property (a locally best choice is always consistent with some globally optimal solution) and optimal substructure (an optimal solution is built from optimal solutions to its subproblems).
  - q: Why does a greedy approach fail on the general 0/1 knapsack problem?
    a: Because taking the item with the best value-to-weight ratio first can use up capacity that an optimal solution would have spent differently, and greedy never revisits that choice once made.
  - q: Which algorithm from the graph-algorithms module is secretly a greedy algorithm, and what choice does it make greedily?
    a: Prim's algorithm greedily adds the cheapest edge that connects a new vertex to the growing tree (Kruskal's algorithm, greedily adding the cheapest edge that doesn't form a cycle, is an equally good answer).
deeper:
  - label: "L-4.1: Introduction to Greedy Techniques With Example"
    url: "https://www.youtube.com/watch?v=v0eQ4nXJjsk"
related_formulas: []
---


## A strategy with no second thoughts

Most of the algorithms covered so far either explore every possibility (brute-force search) or carefully weigh several options before committing (the shortest-path and spanning-tree algorithms, which track running costs across the whole graph). A greedy algorithm takes a different stance: at every step, look at the choices available right now, pick whichever one looks best by some simple local rule, and never go back. No backtracking, no recomputation, no "what if I'd chosen differently three steps ago." That simplicity is what makes greedy algorithms fast — usually a single pass over the input, often with a sort at the start — but it's also exactly what makes them risky. A greedy algorithm is only as good as the guarantee that its local rule never leads it somewhere a smarter algorithm wouldn't have gone.

## When greedy is provably right

Greedy earns that guarantee only when a problem has two properties. The first is the **greedy-choice property**: making the locally optimal choice at a given step is always consistent with *some* globally optimal solution, meaning you never have to undo it later. The second is **optimal substructure**: once that first choice is made, the optimal solution to what remains is itself an optimal solution to a smaller version of the same problem. Together, these mean you can peel the problem down one greedy choice at a time and never paint yourself into a corner.

The classic worked example is **activity selection**: given a set of activities, each with a start and end time, and a single resource (a room, a machine), schedule as many non-overlapping activities as possible. The greedy rule — always pick the next activity that finishes earliest among those compatible with what's already scheduled — is provably optimal. The proof is a short exchange argument: if an optimal schedule didn't pick the earliest-finishing activity first, you could swap it in without reducing the room left over for everything after, so there's always an optimal solution that agrees with the greedy choice.

**Huffman coding**, the variable-length prefix code used for lossless compression, is built the same way: repeatedly merge the two least-frequent symbols into a new node, greedily, and the resulting tree is provably the one that minimizes the expected code length. And the two minimum-spanning-tree algorithms from the graph-algorithms module are greedy algorithms you've already met — Prim's algorithm greedily grows a tree by always adding the cheapest edge that connects a new vertex, and Kruskal's algorithm greedily adds the cheapest edge overall that doesn't create a cycle. In both cases, a classical exchange-argument proof (via the "cut property" of spanning trees) is what licenses the greedy shortcut.

## When greedy quietly fails

The danger is that a greedy algorithm doesn't announce when it's wrong — it just runs to completion and hands back an answer that looks plausible but isn't optimal. The standard counterexample is the **0/1 knapsack problem**: given items with weights and values and a knapsack with a weight capacity, choose a subset of items (each either fully taken or left behind) that maximizes total value without exceeding capacity. The tempting greedy rule is to sort items by value-per-unit-weight and take the best ratio first. Consider a knapsack with capacity 50, and three items: (weight 10, value 60), (weight 20, value 100), (weight 30, value 120). By value density, item 1 is best (6/unit), then item 2 (5/unit), then item 3 (4/unit). Greedy takes item 1 (10 used, value 60), then item 2 (30 used, value 160), and can't fit item 3 (would need 60). Total: 160. But taking items 2 and 3 together uses all 50 of the capacity for a value of 220 — strictly better, and greedy never finds it, because committing to item 1 early used up capacity that a smarter allocation would have spent differently.

This is the tell that a problem lacks optimal substructure in the form greedy needs: a locally best choice can make a *later* choice worse in a way the algorithm can't see and won't undo. (The **fractional** version of knapsack, where items can be split, actually is solved optimally by the same greedy rule — it's specifically the all-or-nothing constraint that breaks it.)

## Recognizing which side of the line you're on

Before reaching for greedy, ask whether you can actually prove the exchange argument: can you show that the locally best choice is always part of *some* optimal solution, the way you can for activity selection, Huffman coding, and minimum spanning trees? If the answer is "it's plausible but I can't show it," that's usually a sign the problem has overlapping subproblems where an early choice can be invalidated by a later one — exactly the situation 0/1 knapsack is in. That's not a dead end; it's the motivation for the next two lessons. Backtracking recovers correctness by being willing to undo a bad choice once it's clearly not working, and dynamic programming recovers it by systematically trying every choice but reusing the answers to overlapping subproblems so that trying everything doesn't mean paying an exponential price every time.
