---
title: Bias and fairness
description: Where bias enters a model and a product built on one, how it is measured, and what an application engineer can actually do about it.
roadmap: ai-engineer
node: bias-and-fairness
group: safety
video:
  provider: youtube
  id: "og67qeTZPYs"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 518
level: intermediate
prerequisites: [guardrails-and-moderation]
takeaways:
  - Bias enters through training data, through labelling, through the objective, and through how a product deploys a model that is working exactly as trained.
  - Fairness has several formal definitions that are mathematically incompatible, so choosing one is a decision about values rather than a technical default.
  - An application engineer rarely controls the weights but controls the inputs, the prompt, the thresholds and whether anyone measured the outcome by group.
check_questions:
  - q: Why can a system not satisfy every fairness definition at once?
    a: Because they make conflicting demands. Equal error rates across groups and equal treatment of equally qualified individuals cannot generally both hold when base rates differ, which is a proven result, not a tuning problem.
  - q: What can you do about bias if you did not train the model?
    a: Measure outcomes by group rather than in aggregate, control what goes into the prompt, examine whether retrieval over-represents some sources, set thresholds deliberately, and keep a person in the loop where the decision affects someone materially.
deeper:
  - label: "MIT 6.S191 — AI Bias and Fairness"
    url: https://www.youtube.com/watch?v=wmyVODy_WD8
  - label: "A Data Odyssey — Formal definitions of fairness in machine learning"
    url: https://www.youtube.com/watch?v=c2_ezuAnCrA
related_formulas: []
---

## Where it comes from

A model doesn't invent bias. It learns whatever was already in its training
data and plays it back faithfully. If that text under-represents certain
groups, encodes historical patterns of who tends to hold which job, or simply
contains ten times more English than Swahili, the model learns that pattern
as if it were just how the world is. That's not a flaw in training — it's
training working exactly as designed, on data that was never neutral to begin
with.

Think of it this way: a model is a mirror held up to its training data. If
the picture it reflects is distorted, the distortion isn't in the glass. It's
in whatever stood in front of it.

A product then stacks its own decisions on top of that mirror: which
examples go into a few-shot prompt, which documents a retrieval index
happens to contain, where a classification threshold gets set, who was in
the room when a default behavior was chosen. Here's the part that matters if
you're building the product rather than training the model: you usually
don't control the weights. You do control this second layer, and almost
every lever actually available to you lives there.

## Four kinds of bias worth telling apart

**Representation bias** — a group is underrepresented or stereotypically
portrayed in the training data or retrieval corpus, so the system's quality
or framing differs for that group.

**Allocation bias** — the system's output decides who gets access to a real
resource, like a loan recommendation or a job screening, and that access
splits along group lines with no legitimate reason behind it.

**Stereotyping** — the output reinforces a cultural stereotype even with no
allocation decision involved at all, like assuming a nurse is "she" by
default.

**Quality-of-service disparity** — the system performs measurably worse for
some groups on the identical task. A speech-to-text system that mishears
some accents far more than others is the textbook case.

## Measurement techniques that actually surface these

**Disaggregated evaluation** reports accuracy by group instead of as one
blended number. A 95% overall accuracy figure can be hiding a group sitting
at 60% — that's exactly what an aggregate number is good at hiding.

**Counterfactual testing** swaps one attribute in an otherwise identical
input — a name associated with one demographic group for a name associated
with another — and checks whether the output changes. It shouldn't. When it
does, you've found something real, and this is one of the cheapest, most
revealing tests you can run.

## Personalization pulls against fairness

A system built to personalize deliberately treats people differently based
on what it knows about them. Fairness, in one common sense of the word, asks
for the opposite — treat people the same regardless of group. These two
goals are in genuine tension, and there's no universal answer for which one
should win.

A recommendation engine personalizing content sits in a very different place
on this question than a system influencing credit or employment decisions.
How much differentiated treatment is acceptable should be a decision someone
made on purpose, not an accident of how the system happened to get built.

## Mitigation beyond measurement

A few concrete levers exist once a disparity has actually been measured.
Prompt instructions asking the model to apply a consistent standard
regardless of identifying details in the input help somewhat, though they
don't fix things on their own. Curating a retrieval corpus for balanced
coverage across the perspectives a system's answers touch on reduces how
much representation bias flows through to what the model reads. Checking
output against a disparity metric before returning it, and regenerating when
the check fails, catches what generation alone missed. None of these works
alone — in practice you stack them.

## What you can actually do day to day

Measure outcomes by group, never only in aggregate — an overall accuracy
number reliably hides the concentrated failures that matter most. Look
directly at a RAG system's retrieval corpus for whose voice and perspective
it actually represents, since that corpus is the product's effective point of
view on anything it covers. Test explicitly with names and dialects outside
whatever the system's unstated default assumption is: a striking share of
real failures surface just by changing a name in an otherwise identical
prompt. Keep a person genuinely in the loop for any decision that materially
affects someone's access to something that matters. And write down what you
decided and why — an unrecorded fairness choice gets silently, inconsistently
re-made the next time someone else touches the same threshold or prompt.
