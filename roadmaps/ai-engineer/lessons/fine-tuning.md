---
title: Fine-tuning
description: Full fine-tuning against LoRA, what adaptation teaches and what it does not, and the real cost of owning a tuned model.
roadmap: ai-engineer
node: fine-tuning
group: production
video:
  provider: youtube
  id: "uLrOI65XbDw"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1072
level: advanced
prerequisites: [rag-vs-fine-tuning, choosing-a-model]
takeaways:
  - Fine-tuning reliably teaches form, style and task shape. It is a poor mechanism for installing facts.
  - LoRA trains a small number of added parameters instead of all of them, which brings adaptation within reach of ordinary hardware and makes adapters swappable.
  - A tuned model is an asset you now own — a dataset to maintain, a base model that will move on, and evaluation that cannot be outsourced.
check_questions:
  - q: What does LoRA change about the training process?
    a: It freezes the original weights and trains small low-rank matrices alongside them, so only a tiny fraction of parameters get updated. Memory drops enormously, and the result is a small adapter rather than a new copy of the model.
  - q: Why is a fine-tuned model a maintenance commitment rather than a one-off?
    a: The dataset has to stay current, the base model it was built on will eventually get deprecated, and every behavior you tuned in has to be re-verified after any change. You've taken ownership of a component you used to rent.
deeper:
  - label: "IBM Technology — Is fine-tuning still needed? LLMs, RAG and LoRA"
    url: https://www.youtube.com/watch?v=-W2JdSl1v48
related_formulas: [cross-entropy, adam]
---

## What fine-tuning is actually for

Fine-tuning takes a pretrained model and keeps training it, this time on your
own examples, nudging its weights toward a specific behavior. It's excellent
at teaching *form* — a consistent output structure, a particular house voice,
a narrow classification distinction, a tool-use pattern the base model keeps
getting subtly wrong. It's a poor way to teach *facts*. The
[rag-vs-fine-tuning](/roadmaps/ai-engineer/rag-vs-fine-tuning) lesson covers
why in depth, but the short version: training bakes in a general tendency, not
a lookup table of specific truths.

The most economically interesting use isn't making a model smarter in some
general sense. It's making a much smaller model good enough for one specific
job. A tuned small model that matches a frontier model's quality on your one
narrow task can cut cost and latency by an order of magnitude — a result
worth real engineering effort, in a way "teach it our internal docs" almost
never is.

## Full fine-tuning, LoRA, and QLoRA

**Full fine-tuning** updates every parameter in the model. It's the most
expressive option, and the most expensive: you need memory not just for the
weights, but for the gradients and the optimizer's own state, which together
typically run several times the model's base size. Beyond a genuinely small
model, that means serious, often multi-GPU hardware.

**LoRA** (low-rank adaptation) takes a different approach. It freezes the
pretrained weights entirely and inserts small additional matrices alongside
them — a tiny fraction of a percent of the total parameter count — and trains
only those. Memory requirements collapse. More importantly, training now
produces a small, separate adapter file instead of a whole new copy of the
model. One base model deployed once can serve many different tuned behaviors
just by swapping which adapter is loaded alongside it — useful if you're
serving many customers, each with their own narrow tuned behavior, off a
single shared base.

**QLoRA** pushes further: it quantizes the frozen base model down to 4-bit
precision while still training the LoRA adapter at higher precision, bringing
useful fine-tuning within reach of a single consumer GPU.

For nearly everything in this roadmap, LoRA is the sensible starting point.
Reach for full fine-tuning only once a measured comparison has actually shown
the adapter approach to be the limiting factor.

## Other training techniques, briefly

Supervised fine-tuning on input-output pairs is the common starting point,
but it isn't the only technique in use. **RLHF** (reinforcement learning from
human feedback) trains on comparisons between pairs of the model's own
outputs, rated by human preference, rather than one fixed correct answer per
input — closer to how providers refine a model's general instruction-following
than how narrow task-specific tuning is usually done. **DPO** (direct
preference optimization) reaches a similar outcome through a simpler,
directly optimizable objective computed straight from preference pairs,
skipping the separate reward model RLHF needs.

These are worth recognizing by name. Application-level fine-tuning for a
specific task is overwhelmingly plain supervised fine-tuning on curated
examples, not either of these.

## The data flywheel — and its dark side

Production traces, the real inputs your system has already handled along
with the corrected or approved outputs, are often the best possible source of
fine-tuning data. They reflect the actual distribution of inputs the tuned
model will face, not a hand-curated and inevitably narrower set. This creates
a real flywheel: production use generates data, the data improves the tuned
model, the improved model handles production better, and better handling
generates still more useful data.

The dark side is **catastrophic forgetting**. Train a model hard toward one
narrow behavior, and it can measurably degrade on other, unrelated tasks it
used to handle fine — intensive training toward one objective can simply
overwrite what the model had learned elsewhere. The fix is an eval set that
covers what you *didn't* mean to change, not just what you did. That's what
catches forgetting before it reaches production.

## Measure the baseline before you start

Hold out a genuine test set before any tuning begins, and measure the
untouched base model's performance on it first. That's the comparison that
actually tells you whether fine-tuning helped at all. A surprising number of
fine-tuning efforts skip this step, and end up unable to say what all that
effort actually bought them — beyond a vague sense that the outputs look
better.

## Quality beats quantity, consistently

A thousand carefully checked, genuinely representative examples routinely
beat fifty thousand scraped together with less care. The model learns the
systematic mistakes in your training set exactly as precisely as it learns
the behavior you meant to teach. Every consistent error pattern in your
examples becomes a learned, consistent behavior in the tuned model.
