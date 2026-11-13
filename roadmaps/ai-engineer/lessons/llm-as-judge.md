---
title: LLM as judge
description: Grading output with a model, aligning that judge against human labels, and the biases that make an unvalidated judge worse than none.
roadmap: ai-engineer
node: llm-as-judge
group: evaluation
video:
  provider: youtube
  id: "kP_aaFnXLmY"
  start: 0
  end: null
creator:
  name: Evidently AI
  channel_url: https://www.youtube.com/@EvidentlyAI
duration_seconds: 1463
level: advanced
prerequisites: [building-an-eval-set]
takeaways:
  - A judge is a model with a rubric, and it's itself a system that needs evaluating before its scores mean anything.
  - Alignment is measured by comparing the judge against human labels on the same cases, and reported as agreement — never assumed.
  - Judges carry systematic biases — position, verbosity, self-preference — that produce confident scores pointing the wrong way.
check_questions:
  - q: How do you know whether a judge is any good?
    a: Label a sample by hand, run the judge on the same cases, and measure agreement. If the judge and a human disagree often, its scores are noise, no matter how reasonable its explanations read.
  - q: What is verbosity bias?
    a: The tendency to score longer answers higher regardless of quality. It's why a judge can report steady improvement while a prompt change is only making the output wordier.
deeper:
  - label: "Eugene Yan — Using LLMs as judges, pairwise grading and the bias catalogue"
    url: https://www.youtube.com/watch?v=7EGF0Mc0_os
  - label: "AI Engineer — Judge the judge, building evaluators that work"
    url: https://www.youtube.com/watch?v=X4dEHRzBLmc
related_formulas: []
---

## A measuring instrument needs calibrating

Some qualities can't be checked by code, no matter how clever the assertion.
Is an answer actually grounded in its cited sources? Does the tone match a
brand guideline? Did a summary keep the genuinely important parts of a
longer document? These need real judgment, and at any real volume that
judgment has to be automated to be practical at all.

Here's the trap: treating a judge's output as a trustworthy measurement just
because it arrives as a tidy number. A judge is another LLM-based system,
with its own prompt and its own failure modes. It inherits nearly every
weakness covered elsewhere in this roadmap. An unvalidated judge produces a
confident-looking number with an unmeasured relationship to actual quality —
which is worse than no number at all, because teams will act on a number that
looks authoritative.

## Reference-based versus reference-free grading

A **reference-based** judge compares a candidate answer against a known-good
reference and scores how closely they match in meaning. Useful when a
genuinely correct answer can be written down in advance for each test case.

A **reference-free** judge grades a candidate purely against a stated
rubric, with no single correct answer to check against. This is what you
need for open-ended tasks — "write a helpful response to this support
ticket" has no one right phrasing, so no reference text could capture it.
Reference-free grading is more broadly applicable, and correspondingly
harder to calibrate, since there's no ground truth to check the judge's
reasoning against.

## Aligning a judge, as a concrete procedure

Label a representative sample yourself — on the order of a hundred outputs,
graded by someone who genuinely understands what a good answer looks like
for this task. Run the candidate judge over that same hundred. Compare the
two sets of labels directly.

Wherever they disagree, read those cases closely. More often than not, the
disagreement is a sign the rubric itself is ambiguous on that case, not that
the judge reasoned incorrectly. Tightening the rubric — typically by adding a
worked example of each grade level into the judge's own prompt, since the
judge is, after all, just another model responding to a prompt and benefits
from the same technique in
[few-shot prompting](/roadmaps/ai-engineer/few-shot-prompting) — usually
resolves most of it.

Once agreement is high enough to trust, keep those original human labels as
the judge's permanent regression set. Any time the judge's prompt or
underlying model changes, re-run it against that same labeled set. That tells
you immediately whether the judge still agrees with the humans, or has
quietly drifted.

## The specific biases to design around

**Position bias**: in a pairwise comparison, the judge measurably favors
whichever position — first or second — it was shown, independent of actual
content. The standard fix: run the comparison in both orders, and either
average the result or treat a disagreement between the two orders as a tie.

**Verbosity bias**: a longer answer tends to score higher from a judge
regardless of whether the extra length added anything. A prompt change that
only makes output more verbose, with no real improvement in correctness, can
show up as a rising judge score — watch for that specifically whenever a
score improves alongside a noticeable jump in output length.

**Self-preference bias**: a judge measurably prefers text from its own model
family over text from a different one, even at genuinely comparable quality.
Using a different model as the judge than the one being evaluated is cheap
insurance against this.

**Scale compression**: ask a judge for a rating on a wide numeric scale, like
one to ten, and you tend to get a narrow cluster of sevens and eights
regardless of true quality spread — the judge has little basis for fine
discrimination along such a wide, under-specified scale. Narrow, precisely
defined scales, with a description of what each point means, carry far more
usable signal. Better still: binary pass-or-fail questions against one
specific criterion.

## Ensembling, and what it buys

Run several independent judge calls — different prompts, different models,
or the same judge at non-zero temperature multiple times — and combine the
verdicts by majority vote or averaging. This dilutes any one judge's
idiosyncratic bias on a given case, at the direct cost of multiplying the
number of calls per evaluated output. It's usually reserved for the
highest-stakes evaluations, like a release gate blocking a deployment,
rather than routine day-to-day iteration, where a single well-calibrated
judge is normally fast and cheap enough to use constantly.

## Prefer narrow, binary questions over holistic ratings

"Does every factual claim in this answer appear somewhere in the provided
source material?" produces a genuinely usable, checkable judgment. "Rate the
overall quality of this answer from one to ten" produces, more often than
not, a seven — and tells you almost nothing actionable about what to fix.
