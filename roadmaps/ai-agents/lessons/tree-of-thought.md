---
title: Tree of thought
description: "Exploring several candidate reasoning paths at once, scoring and pruning them, instead of committing to one linear chain from the start."
roadmap: ai-agents
node: tree-of-thought
group: reasoning-and-planning
video:
  provider: youtube
  id: "zwfE6J2BIR4"
  start: 0
  end: null
creator:
  name: Shunyu Yao
  channel_url: https://www.youtube.com/@shunyuyao4854
duration_seconds: 3600
level: intermediate
prerequisites: [react-reason-and-act]
takeaways:
  - Tree of thought explores multiple candidate reasoning paths as branches of a tree, rather than committing to one linear chain, which matters on problems with no single obvious next step.
  - Each partial path is evaluated for how promising it looks before the agent commits further resources to extending it, which is what makes pruning and backtracking possible.
  - Backtracking away from a weak branch and returning to try an alternative is the capability a single linear chain structurally cannot offer, since a chain has no earlier branch point to return to.
  - The approach costs more computation than a single chain, because multiple partial paths are generated and scored, so it is reserved for problems where no single next step is clearly correct.
check_questions:
  - q: What kind of problem benefits most from tree of thought rather than a single reasoning chain, and why?
    a: Problems with no single obviously-correct next step, such as puzzles or planning tasks, benefit most, because a single chain commits irreversibly to one guess at each step while a tree can generate several candidate next steps, score them, and abandon the ones that turn out weak.
  - q: Why does tree of thought require more computation than chain-of-thought or ReAct?
    a: Because at each branching point it generates and evaluates multiple candidate continuations rather than just one, so the total work scales with the number of branches explored, not just the length of the eventual path actually used.
deeper: []
related_formulas: []
---

## When one chain is not enough

ReAct and chain-of-thought both produce a single line of reasoning: one thought leads to one action leads to one next thought, in sequence. That works well when each step has an obviously correct next move. It works poorly when a problem genuinely branches — when, at some point, there are two or three plausible next steps and no way to tell in advance which one leads somewhere useful. Committing to one guess and running it all the way to a dead end wastes the entire chain; there is no earlier point to return to, because a chain has no stored alternatives. Tree of thought addresses exactly this by keeping several candidate continuations alive at once, evaluating how promising each looks, and discarding the weak ones before they are extended any further — turning the single committed guess into a search over several.

## The mechanism: branch, score, prune, continue

At each step with more than one plausible move, the agent generates several candidate next steps rather than one. Each candidate is then evaluated — either by the model itself judging how promising the partial path looks, or by a simple checkable rule for how close it gets to the goal. Candidates that score poorly are pruned: the agent stops extending them and does not spend further computation on them. The agent continues extending the surviving, more promising candidates, repeating the branch-score-prune cycle at each subsequent decision point. If every surviving path from an early branch eventually dead-ends, the agent backtracks — returns to an earlier branch point and tries a different candidate that was set aside rather than being forced to start the whole problem over.

## A worked example: Game of 24

In Game of 24, you are given four numbers and must combine them with +, −, ×, ÷ to reach exactly 24. Take the numbers 4, 7, 8, 8.

**Branch 1 (first step):** the agent considers several first moves: 8 × 8 = 64, 8 + 8 = 16, 7 − 4 = 3, 8 ÷ 4 = 2.

**Score:** 16 (from 8+8, leaving 4, 7) looks promising because 16 is close to 24 and the remaining numbers 4 and 7 could plausibly adjust it. 64 is pruned immediately — it overshoots badly and the remaining numbers 4, 7 can't bring 64 down to 24 cleanly. 3 and 2 are kept as weaker backups.

**Branch 2 (extend the 16 branch):** from 16 with remaining numbers {4, 7}, the agent tries 16 + 7 = 23 and 16 + 4 = 20 — both close to 24 but neither reachable exactly with the one number left over in each case. This branch is pruned: it cannot resolve.

**Backtrack:** the agent returns to the branch point and tries a candidate set aside earlier — 8 ÷ 8 = 1, remaining numbers {4, 7, 1}. Scored as promising because 1 is a flexible number to combine with 7.

**Branch 3 (extend the 8÷8=1 branch):** from remaining {4, 7, 1}, try 7 − 1 = 6, leaving {4, 6}. Scored highly: 6 × 4 = 24 exactly.

**Resolution:** the full path is (7 − 8 ÷ 8) × 4 = 24, using all four original numbers exactly once. The agent only found this by abandoning the 16-branch once it stalled and returning to try 8 ÷ 8 = 1 instead — a move the tree kept on record as a backup candidate rather than discarding the moment the first branch looked promising.

The point of the walkthrough is the shape, not the arithmetic: several first moves tried, the clearly bad one (8×8=64) pruned immediately, the next-most-promising one (8+8=16) extended and then abandoned when it stalled, and a backtrack to a previously-set-aside candidate that turned out to be the winning path. A single linear chain gets exactly one guess at the first move and has no way back if it's wrong; the tree keeps options open and gets to try again.

## Why this costs more, and when that's worth it

Generating and scoring multiple candidates at every branch point means tree of thought does strictly more work than a single chain — if a chain takes one pass per step, a tree with three candidates per branch does roughly three times the generation and all the scoring on top. That cost is wasted on problems that don't actually branch, which is why chain-of-thought and ReAct remain the default for sequential, mostly-linear tasks. Tree of thought earns its keep specifically on puzzles, planning problems, and creative tasks where an early wrong turn is expensive to discover late and cheap to avoid early — paying more compute upfront to avoid committing to a plan that only reveals its flaw several steps downstream.
