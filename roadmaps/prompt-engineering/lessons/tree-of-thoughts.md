---
title: Tree of thoughts
description: Branch into multiple partial solutions, score them, and search the tree — for problems one straight-line reasoning chain can't solve.
roadmap: prompt-engineering
node: tree-of-thoughts
group: advanced-patterns
video:
  provider: youtube
  id: "MXBxNUnC7iY"
  start: 0
  end: null
creator:
  name: Lucas Soares (Automata Learning Lab)
  channel_url: https://www.youtube.com/@automatalearninglab
duration_seconds: 495
level: intermediate
prerequisites: [chain-of-thought-prompting]
takeaways:
  - Tree of thoughts generates multiple candidate next steps at each stage of reasoning, scores them, and searches the resulting tree rather than committing to one linear path.
  - Pruning weak branches early and backtracking when a path stalls is what a single chain-of-thought or independent self-consistency samples cannot do.
  - The extra search machinery is worth its cost only on problems with distinguishable intermediate states and real decision points — not on tasks with one obvious path through.
check_questions:
  - q: How does tree of thoughts differ from generating several independent chains of thought and voting, as self-consistency does?
    a: Self-consistency runs complete, independent reasoning chains from start to finish and votes on the final answers. Tree of thoughts builds one shared tree, evaluates partial paths as it goes, and can abandon a weak branch partway through or backtrack to an earlier decision point — something that requires full, separate runs to do.
  - q: What two extra ingredients does tree of thoughts add on top of chain-of-thought prompting?
    a: A way to generate multiple candidate next steps at each point in the reasoning rather than just one, and a way to evaluate or score partial paths so a search procedure — breadth-first or depth-first — can decide which branches to keep exploring and which to prune.
deeper:
  - label: "Yannic Kilcher — Tree of Thoughts: Deliberate Problem Solving with LLMs (Full Paper Review)"
    url: https://www.youtube.com/watch?v=ut5kp56wW_4
related_formulas: []
---

## A tree instead of a line

Chain-of-thought prompting produces one linear sequence of reasoning steps
from question to answer. Self-consistency produces several such sequences,
generated independently, and votes on the final answers. Tree of thoughts,
introduced by Yao et al. in 2023, does something structurally different: it
builds a single tree of partial reasoning steps, where each node is a
partial solution, each node can branch into multiple candidate next steps,
and a search procedure decides which branches are worth continuing and
which to abandon before they're finished.

This matters for a specific class of problem that linear reasoning and
independent sampling both handle poorly: problems where an early decision
can be locally reasonable but turn out to be a dead end several steps
later, and where recognizing that requires comparing partial progress, not
just final answers. A single chain-of-thought run has no way to notice it's
heading toward a dead end and change course. Self-consistency runs several
full attempts independently and only compares them at the very end — it
can't abandon a bad path early, because by the time you see the final
answer, all the computation on that path has already been spent.

## The four pieces of the method

Tree of thoughts is built from four components working together. First,
**thought decomposition**: breaking the overall problem into a sequence of
intermediate steps small enough that "a partial solution so far" is a
meaningful, comparable unit — for a word puzzle this might be one placed
piece; for a math problem, one derived intermediate quantity. Second,
**thought generation**: at each node in the tree, prompting the model to
propose several different candidate next steps rather than just one, so the
tree actually branches. Third, **state evaluation**: a separate prompt (or
the same model acting in a different role) looks at a partial path so far
and estimates how promising it is — plausible, unlikely to lead anywhere,
or already clearly wrong. Fourth, a **search algorithm** — breadth-first
search, which keeps the best few candidates at each depth and expands them
in parallel, or depth-first search, which pushes one promising branch as
deep as it goes and backtracks when it evaluates poorly — that uses those
evaluations to decide where to spend further computation.

## A concrete shape: the Game of 24

The paper's running example is the Game of 24: given four numbers, combine
them with addition, subtraction, multiplication, and division to reach
exactly 24. A single chain-of-thought attempt picks one sequence of
operations and either hits 24 or doesn't, with no way to recover from an
early unproductive choice. Tree of thoughts instead generates several
candidate first operations (combine the first two numbers in a few
different ways), evaluates which resulting three-number states still look
reachable to 24, keeps the promising ones, and repeats — branching again
from each surviving state, pruning ones that evaluate as unlikely to reach
24, and backtracking to try a different second-level operation if a branch
that looked promising turns out to dead-end. The structure generalizes
to any problem with the same shape: a sequence of choices, intermediate
states that can be judged as more or less promising before the problem is
fully solved, and a real risk that greedy, uncorrected linear reasoning
walks into a dead end.

## When the machinery is worth it

Tree of thoughts costs considerably more than chain-of-thought or even
self-consistency — it requires many more model calls to generate branches,
evaluate states, and search, plus the implementation overhead of actually
running the search procedure rather than just parsing a single response.
That cost is justified only when a problem has two properties at once:
genuine decision points where different choices lead down meaningfully
different paths, and intermediate states that can be evaluated as more or
less promising before the problem is fully solved. Planning problems,
puzzles with branching move trees, and multi-step problems where an early
wrong turn is easy to make but hard to notice without comparing partial
progress are good fits. Straightforward factual questions, simple
classification, or tasks with one obvious reasoning path gain nothing from
the extra structure — there's no branching to search in the first place,
so you're paying for machinery the problem doesn't need.

## Positioning it against the rest of this roadmap

It's worth being precise about where tree of thoughts sits relative to the
other techniques in this group. Self-consistency assumes a single good
*strategy* exists and that running it several times averages out random
noise in execution. Tree of thoughts assumes the *strategy itself* has
genuine branch points worth exploring and comparing, and adds the machinery
to do that exploration deliberately rather than by chance. In practice, the
two can combine — sampling multiple candidates at each node is itself a
small act of self-consistency, applied locally within the tree rather than
across whole independent runs.
