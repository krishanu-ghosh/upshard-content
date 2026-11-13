---
title: Fine-tuning vs. prompting
description: When prompting hits its ceiling, what fine-tuning actually buys you, and a decision rule for choosing between them.
roadmap: prompt-engineering
node: fine-tuning-vs-prompting
group: optimization
video:
  provider: youtube
  id: "zYGDpG-pTho"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 790
level: intermediate
prerequisites: [choosing-a-model]
takeaways:
  - Prompting changes nothing about the model; it is fast to iterate but bounded by context length and can't reliably change behavior patterns across many interactions.
  - Fine-tuning updates the model's own weights on your examples, which is better for consistent style, format, and domain adaptation at scale, at the cost of a real training dataset and slower iteration.
  - Start with prompting, and move to fine-tuning only once prompting's ceiling is reached and there are enough quality examples to train on.
check_questions:
  - q: Why can prompting struggle to produce consistent behavior across a large number of varied interactions, even with a very good prompt?
    a: A prompt only steers a single call's output within that call's context; it does not change the model's underlying weights, so edge cases outside what the instructions anticipated fall back on the base model's default behavior rather than the trained-in pattern fine-tuning would produce.
  - q: What does fine-tuning require that prompting does not?
    a: A labeled training dataset of real examples, infrastructure to run the training job, and time to iterate — none of which prompting needs, since prompting only edits the text sent at inference time.
deeper: []
related_formulas: []
---

## The same question, asked a different way each time

Every lesson so far in this roadmap has treated the model as fixed and the prompt as
the only thing you control. That is prompting's whole premise: the model's weights
never change, so every improvement has to come from what you say and how you say it.
Fine-tuning rejects that premise — it changes the model itself, by continuing its
training on your own examples. The two are not competing techniques so much as two
different places to intervene, and knowing which one a problem actually calls for is
worth more than getting better at either one in isolation.

A brief note on a third option that's easy to conflate with these two: retrieval-
augmented generation (RAG) changes neither the model nor a fixed prompt — it changes
what information is available to the model, by retrieving relevant documents at query
time and inserting them into the context. RAG solves a knowledge problem (the model
doesn't know this fact, or this fact is private/current and wasn't in training data).
It doesn't solve a behavior problem (the model knows the facts but won't format, phrase,
or reason about them the way you need), which is the actual fine-tuning-vs-prompting
question this lesson is about.

## What prompting can't fix, no matter how good it gets

A prompt works by conditioning a single generation: given this context, produce this
output. It is fast — editing text and re-running costs nothing but time — and it
requires no training data, no infrastructure, and no waiting. But every one of its
strengths traces back to the same limitation: it only ever shapes one call at a time,
within whatever fits in the context window. Three kinds of problems sit past that
ceiling.

First, consistency at scale: a prompt that handles the ten cases you tested can still
fail on the hundredth edge case nobody thought to specify, because the model's default
behavior underneath the instructions hasn't changed — only this call's framing has.
Second, format and style baked in deeply: getting a model to reliably adopt a very
specific voice, structure, or domain convention across thousands of varied inputs often
needs more consistent reinforcement than any single instruction block provides. Third,
prompt length and cost: if achieving reliable behavior requires an increasingly long
system prompt with more and more examples and edge-case handling, that's a sign the
behavior itself needs to be trained in rather than re-explained on every single call.

## What fine-tuning actually changes

Fine-tuning takes a pretrained model and continues training it — updating its actual
weights — on a dataset of examples specific to your task. Instead of describing the
desired behavior in a prompt every time, you show the model hundreds or thousands of
examples of that behavior and let gradient descent adjust the weights to produce it by
default. The result is a model that exhibits the trained pattern without needing it
re-explained in the prompt at all, which also shortens every subsequent call — no more
paying token cost for a long instruction block repeated on every request.

The price is real: a labeled dataset good enough to train on (garbage examples produce
a model that's confidently wrong in a new way, not a fixed one), infrastructure or a
provider's fine-tuning API to run the job, and a slower iteration loop — a prompt edit
takes seconds to test, a fine-tuning run takes a training job's worth of time. Fine-
tuning also doesn't compose the way prompting does: changing the instructions is one
edit, but changing trained-in behavior means a new training run.

## A decision rule worth actually following

Start with prompting, always — it is nearly free to try and tells you quickly whether
the problem is even solvable by better instructions. Move to fine-tuning only when two
things are both true: prompting has genuinely hit its ceiling (not just "the first
prompt didn't work," but real iteration has stalled), and there is an actual dataset of
quality examples — not a hypothetical one — to train on. Fine-tuning without good
data doesn't rescue a vague prompting strategy; it just encodes the same vagueness more
expensively and more permanently. The lessons earlier in this roadmap on
[evaluation and testing](/roadmaps/prompt-engineering/evaluation-and-testing) are exactly
what tells you, with evidence rather than a hunch, whether you've actually reached that
ceiling.
