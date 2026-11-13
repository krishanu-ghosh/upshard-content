---
title: Self-consistency prompting
description: Sample several independent reasoning paths and take the answer most of them agree on, instead of trusting just one.
roadmap: prompt-engineering
node: self-consistency-prompting
group: advanced-patterns
video:
  provider: youtube
  id: "K8zcj_i7cVc"
  start: 0
  end: null
creator:
  name: Dhaval Maheta (DM)
  channel_url: https://www.youtube.com/@DhavalSaifaleeAaryash
duration_seconds: 496
level: intermediate
prerequisites: []
takeaways:
  - Self-consistency samples a chain-of-thought prompt several times at temperature greater than zero and takes the most common final answer.
  - It works because independent reasoning attempts tend to fail in different, uncorrelated ways, so the correct answer is the one multiple attempts converge on.
  - The cost is literal — N times the inference calls and latency for one answer — so it's worth reserving for hard or high-stakes questions, not routine ones.
check_questions:
  - q: What does self-consistency do differently from a single chain-of-thought call?
    a: It runs the same chain-of-thought prompt multiple times with randomness enabled, producing several independent reasoning paths, then picks the final answer that appears most often across them rather than trusting the one path from a single call.
  - q: Why does taking a majority vote across reasoning paths improve accuracy?
    a: Individual reasoning chains tend to go wrong in different, largely uncorrelated ways, so a wrong answer is unlikely to be the one most paths arrive at independently — while a genuinely correct line of reasoning is more likely to recur across multiple attempts.
deeper:
  - label: "Analytics Vidhya — Chain of Thought Prompting | Self Consistency Prompting Explained"
    url: https://www.youtube.com/watch?v=5-Fv_ZoAwrE
related_formulas: []
---

## One chain of thought is one roll of the dice

Chain-of-thought prompting — asking a model to reason step by step before
giving a final answer — measurably improves performance on problems that
need multiple steps of reasoning. But it still produces a single reasoning
path per call, and that path can go wrong: a dropped term, a
misread constraint, a plausible-but-incorrect intermediate conclusion that
the rest of the answer builds on without noticing. Chain-of-thought makes
reasoning visible, but it doesn't make any single reasoning attempt reliable
on its own.

Self-consistency, introduced by Wang et al. in 2022, treats this as a
sampling problem rather than a prompting problem. Instead of asking for one
chain of thought and trusting it, you ask for the same chain-of-thought
prompt multiple times — typically somewhere between five and twenty times —
with the model's sampling temperature set above zero so each run can diverge.
Each run produces its own reasoning path and its own final answer. You then
look across all the final answers and take whichever one appears most often:
a majority vote over independently generated attempts at the same problem.

## Why voting beats trusting the first answer

The method rests on a simple empirical observation: when a model makes a
mistake while reasoning through a problem, the mistake tends to be different
from run to run. One attempt might misapply a formula; another might
misread which quantity the question is actually asking for; a third might
make an arithmetic slip midway through. These errors are not perfectly
correlated with each other, which means that across several independent
attempts, incorrect answers tend to scatter across multiple different wrong
values, while a correct line of reasoning — derived the same valid way each
time — tends to land on the same correct answer repeatedly. The majority
answer ends up disproportionately likely to be the correct one, even
though no individual run was verified or checked.

Concretely: ask a model a multi-step word problem five times at a moderate
temperature. Suppose three runs arrive at 42, one arrives at 39, and one
arrives at 51. Self-consistency takes 42 — not because any one of those
runs was checked for correctness, but because three independent reasoning
attempts converged on it while the other two each went wrong in their own
distinct way. This is the same intuition behind ensemble methods in
traditional machine learning: independent, imperfect estimators can combine
into a more reliable one, provided their individual errors aren't all
pointing in the same direction.

## The relationship to chain-of-thought

Self-consistency is not a replacement for chain-of-thought — it's built on
top of it. Each of the sampled runs is itself a full chain-of-thought
prompt, reasoning through the problem step by step before producing an
answer. What self-consistency adds is the layer above those individual
runs: generate several of them, then aggregate. If you strip the sampling
and voting away, you're left with ordinary chain-of-thought prompting,
which is why this technique only makes sense once you already have a
chain-of-thought prompt worth running multiple times.

## The real cost, and when to pay it

The honest cost of self-consistency is that it's literally N times the
inference calls for one answer, where N is however many samples you draw.
That's N times the token spend and, unless you parallelize the calls, N
times the latency. This isn't a technique to apply by default to every
prompt in a product — it's a technique to reach for when a single wrong
answer is expensive and the problem is genuinely the kind where reasoning
can plausibly go wrong in more than one way. A multi-step math or logic
problem, a question with several plausible-looking wrong answers, or a
decision that's costly to get wrong downstream are good candidates. A
simple factual lookup, a classification with an obvious answer, or a task
where chain-of-thought alone already performs near-ceiling accuracy are not
— the extra calls buy you very little there, and the latency and cost are
pure overhead.

## Extracting the answer to vote on

One practical wrinkle: voting requires being able to compare final answers
across runs, which means the prompt needs to produce an answer in a
consistent, extractable form — a single number, a single label, a short
final line — rather than free-form prose that differs in wording between
runs even when the underlying answer agrees. A common pattern is to end the
chain-of-thought prompt with an explicit instruction like "state your final
answer on its own line, in the form `Answer: <value>`," which makes the
values trivial to pull out and tally programmatically once all the samples
are back.
