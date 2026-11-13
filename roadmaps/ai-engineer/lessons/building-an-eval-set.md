---
title: Building an eval set
description: Sourcing real failures, labelling them, writing assertions that do not need a model, and keeping the set from going stale.
roadmap: ai-engineer
node: building-an-eval-set
group: evaluation
video:
  provider: youtube
  id: "tqUDjc1HzO4"
  start: 0
  end: null
creator:
  name: Hamel Husain
  channel_url: https://www.youtube.com/@hamelhusain7140
duration_seconds: 1639
level: intermediate
prerequisites: [why-evals-matter]
takeaways:
  - The best eval cases come from production logs, because invented inputs test the failures you imagined rather than the ones you have.
  - Prefer a deterministic check wherever one exists — cheap, fast, unambiguous — and reserve model grading for what genuinely needs judgement.
  - Split the set into one you iterate against and one you do not look at, or you will tune until the number is good and learn nothing.
check_questions:
  - q: Why keep a portion of the eval set unexamined?
    a: Because repeatedly tuning against the same cases fits them specifically. A held-out set that you only run occasionally is the one that tells you whether the improvement generalises.
  - q: What makes a deterministic assertion preferable to a model judge where both would work?
    a: It is free, instant and unarguable. Valid JSON, a required citation, a forbidden phrase and a length bound are all checkable in code, and every such check removes a judgement call.
deeper:
  - label: "Dave Ebbelaar — Systematically setting up LLM evals, levels and unit tests"
    url: https://www.youtube.com/watch?v=a3SMraZWNNs
  - label: "Arize AI — Building a golden dataset end to end"
    url: https://www.youtube.com/watch?v=NaCjm8rdxqk
related_formulas: []
---

## Where the cases come from

Pull them from production, as far as that's genuinely possible. Real inputs
carry the typos, the ambiguity, the language mixing, and the adversarial
edge cases that actually break a system. Invented inputs only carry the
assumptions of whoever invented them about what failure looks like — a
strictly smaller, more comfortable set than reality ever provides. It's the
difference between testing a login form with the inputs you designed it for,
and testing it with what a real user actually typed, half-asleep, on a phone
with autocorrect fighting them the whole way.

If there's no production traffic yet, the next best source is anything real
users wrote elsewhere: support tickets, search logs, query logs from an
earlier manual process. Not examples a developer invents from imagination.
Keep the awkward cases on purpose — a set of only clean, well-formed inputs
certifies the system handles the easy half of its job and says nothing about
the half that actually causes incidents.

## Synthetic data, and its real pitfalls

When no real usage exists yet, generating synthetic test cases with a model
is a reasonable stopgap. But it carries a specific risk: a model generating
test inputs tends to generate inputs that are themselves fairly easy for a
model to handle, because the generator and the system under test share
similar blind spots and similar ideas of what a plausible question looks
like. Treat synthetic data as a placeholder to get started, not a long-term
substitute for real usage — replace it with production-derived cases the
moment any real traffic exists.

## Stratified sampling, so coverage isn't accidental

Pull cases at random from production traffic and you'll mostly get whatever
is most common — the easy, high-volume cases — while the rare but important
ones get buried. Stratified sampling pulls a meaningful number of cases from
each known category deliberately: by language, by product area, by user
type, by length. This is how a set stays useful for catching a regression
that only shows up on non-English input, or on an unusually long request,
even though those make up a small slice of overall volume.

## Labelling, and knowing whether your labels can be trusted

Where human judgement is involved, check inter-rater reliability — whether
two people labelling the same case independently tend to agree — rather than
assuming it. If two reasonable people disagree often on what counts as a
good answer, the rubric is ambiguous. That ambiguity shows up later as noisy,
hard-to-interpret eval scores, no matter how sophisticated the rest of the
pipeline is. Tightening the rubric, usually by adding worked examples of what
each grade level looks like, resolves disagreement faster than just asking
labellers to try harder.

## Size, and the trigger for growing it

Start smaller than feels respectable. Fifty to a hundred well-chosen cases
spanning the real, known failure modes beats a thousand near-duplicate easy
ones — if only because a hundred cases can actually be read in full by a
person, and a thousand can't. Grow the set when a genuinely new failure mode
shows up in production that nothing currently in the set would have caught.
That event is the trigger, not a calendar or a round number.

## Splitting, so you don't just fit the examples

Keep two distinct sets. A development set, iterated against freely while
making changes. And a held-out set, run only occasionally and never directly
tuned against, which exists purely to answer whether an improvement measured
on the development set actually generalizes — or whether it was fitted to
the handful of cases you'd been staring at. Without a genuinely held-out set,
the easy failure mode is optimizing until the development score looks good,
then discovering in production that the improvement doesn't transfer,
because you fitted the examples instead of solving the underlying problem.

## Staleness, and keeping the set honest over time

An eval set is a snapshot of the system as it was the moment the set was
built. Prompts change, the model behind the endpoint changes, and the real
users and their requests change right along with both. Re-read a sample of
the eval set on a regular cadence, retire cases that no longer represent
anything meaningful about current usage, and add whatever new failure showed
up since the last review. A set nobody revisits quietly turns into a green
light that has stopped meaning anything about current quality.

## Assert before you judge

Every check you can express in code is a check nobody ever has to argue
about later. Did the output parse as valid JSON in the required format? Does
it cite at least one source when a citation is required? Does it avoid a
phrase a stakeholder flagged as unacceptable? Is it under a stated length
limit? Did it correctly abstain on a case with no real answer? These cost
almost nothing to run, return the same result every time, and in practice
catch a surprising number of real regressions — often more than the
sophisticated, judgement-based grading everyone reaches for first.
Model-based judgement, covered next, is for whatever genuinely can't be
reduced to a deterministic check.
