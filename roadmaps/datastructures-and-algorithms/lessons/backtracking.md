---
title: "Backtracking"
description: Building a solution one piece at a time and abandoning a branch the moment it can't work, worked through the N-Queens problem.
roadmap: datastructures-and-algorithms
node: backtracking
group: algorithm-design-paradigms
video:
  provider: youtube
  id: "DKCbsiDBN6c"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 900
level: intermediate
prerequisites:
  - greedy-algorithms
takeaways:
  - "Backtracking builds a solution incrementally, one decision at a time, and immediately abandons (\"backtracks\" from) any partial solution that can no longer lead to a valid complete one."
  - "It is brute-force search made tractable by pruning — instead of generating every full candidate and checking it at the end, it checks validity after every single decision, so a doomed branch is cut off long before it's fully built."
  - "The N-Queens problem — place N queens on an N×N board so none attacks another — is the standard teaching example: queens are placed column by column, and a column is skipped entirely once a queen there would be attacked by one already placed."
  - "The same place-check-backtrack pattern solves the Knight's Tour (visit every square exactly once with a knight), the Hamiltonian path problem (visit every vertex of a graph exactly once), and maze-solving (find a path from entrance to exit)."
  - "Backtracking explores a search tree whose worst case is still exponential, but pruning typically cuts away the overwhelming majority of that tree, which is what separates it from naive brute force in practice."
check_questions:
  - q: What's the core difference between backtracking and simply generating every possible full candidate and checking each one?
    a: Backtracking checks whether a partial, incomplete solution is still viable after every single decision, and abandons it immediately if not, whereas naive brute force builds every candidate all the way to completion before checking any of them.
  - q: In the N-Queens problem, what does it mean to "backtrack" after placing a queen in a column?
    a: It means removing that queen and trying a different row in the same column once every row has been tried and none led to a solution further along, then returning that failure one level up to try a different placement there too.
  - q: Why is backtracking still worst-case exponential even though it's much faster than naive brute force in practice?
    a: Because the underlying search tree it explores is the same size as brute force's in the worst case — pruning removes branches early rather than shrinking the tree itself, so a problem instance with little to prune can still take exponential time.
deeper:
  - label: "N Queen Problem Using Backtracking Algorithm"
    url: "https://www.youtube.com/watch?v=xouin83ebxE"
related_formulas: []
---


## Trial, error, and knowing when to give up early

The previous lesson's greedy algorithms share one trait: once a choice is made, it's never revisited. That's what makes them fast, and it's also exactly what makes them wrong on problems like 0/1 knapsack, where an early choice can quietly rule out the real answer. Backtracking is built for exactly that situation. It still makes one choice at a time, building a candidate solution piece by piece, but it keeps a safety net: as soon as a partial solution is checked and found to be invalid — or provably unable to lead anywhere valid — the algorithm backs up to the most recent decision point and tries the next option there instead. It's trial and error with a short memory for what's already failed.

This makes backtracking a refinement of brute-force search rather than something fundamentally different from it. Brute force would generate every complete candidate solution and check each one at the end; backtracking checks validity after *every single step* of construction, so an invalid branch is abandoned the moment it becomes invalid rather than after it's been built out to full size. The search space explored is the same tree in principle, but backtracking prunes huge parts of it before ever reaching their leaves.

## Worked example: the N-Queens problem

The textbook example is **N-Queens**: place N chess queens on an N×N board so that no two attack each other (no shared row, column, or diagonal). A brute-force approach might try every way to place N queens on N² squares — astronomically large even for modest N. Backtracking instead places queens one column at a time:

1. Start in column 0. Try row 0. Check: is this square attacked by any queen already placed? (For the first queen, trivially no.) Place it and move to column 1.
2. In column 1, try row 0. Check against every queen placed so far (just the one in column 0) — if it shares a row or diagonal, this placement is invalid; skip to row 1 and check again.
3. Continue down column 1's rows until a valid one is found, place a queen there, and advance to column 2.
4. If *no* row in the current column works — every single one is attacked by a queen placed in an earlier column — that means the choice made in some earlier column was bad. **Backtrack**: remove the queen from the previous column, and resume trying later rows in *that* column where you left off.
5. Repeat until either all N columns have a queen (a solution) or the backtracking unwinds all the way past column 0 with no options left (no solution exists for this N, which only happens for N = 2 and N = 3).

The key move is step 4: the algorithm doesn't wait until all N queens are placed to discover a conflict. It checks after every single placement, so a doomed branch — say, a placement in column 1 that makes columns 2 through 7 all unsolvable — gets cut off at column 1 instead of being discovered only after wastefully trying every possible completion. For N = 8, this pruning is the difference between checking a meaningful fraction of a 64-choose-8 search space and brute-forcing something far larger.

## The same pattern, different boards

Once the place-check-backtrack shape is visible, it shows up everywhere. The **Knight's Tour** problem — move a knight around a board so it visits every square exactly once — uses the identical structure: from the current square, try each legal knight move in some order, recurse into it, and if every move from there eventually dead-ends, backtrack and try the next option from the current square. The **Hamiltonian path problem** — find a path through a graph that visits every vertex exactly once — backtracks over which unvisited neighbor to move to next, abandoning a path the moment it reaches a vertex with no valid unvisited neighbor to continue to. And **maze-solving** is the most literal version: try a direction, recurse, and if that path dead-ends, backtrack to the last junction and try the next untried direction.

## What backtracking does and doesn't buy you

It's worth being honest about the limits here. Backtracking's worst-case time complexity is still exponential — for a maze with no walls at all, or an N-Queens instance that happens to have very few invalid partial placements, the pruning doesn't have much to cut, and the algorithm explores close to the full search tree. What backtracking buys is a *typical-case* speedup, often an enormous one, by refusing to waste time completing a branch that's already doomed. That's a genuinely different trade-off from the greedy algorithms in the previous lesson, which are fast unconditionally but only *correct* when the problem has the right structure. Backtracking is correct unconditionally — it will always find a solution if one exists, because it's really just a pruned exhaustive search — but its speed depends on how much pruning the problem allows. The next lesson picks up a third strategy, dynamic programming, for the specific case where backtracking's branches aren't just large but *repeat* the same subproblems many times over, which opens up a way to avoid redoing that work entirely.
