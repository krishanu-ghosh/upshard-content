---
title: RAG or fine-tuning
description: Which problem each one actually solves, why fine-tune it on our docs is usually the wrong instinct, and when the answer is neither.
roadmap: ai-engineer
node: rag-vs-fine-tuning
group: retrieval
video:
  provider: youtube
  id: "Q-_D_2NWECE"
  start: 0
  end: null
creator:
  name: codebasics
  channel_url: https://www.youtube.com/@codebasics
duration_seconds: 563
level: intermediate
prerequisites: [rag-pipeline]
takeaways:
  - Retrieval supplies facts at query time; fine-tuning teaches form, format and behaviour. Confusing the two is the most common expensive mistake in this field.
  - Fine-tuning on a document corpus does not reliably install those documents as retrievable facts, and it makes updating them a retraining job.
  - A large fraction of problems that look like either are solved by a better prompt, better context, or a more capable model.
check_questions:
  - q: Why is fine-tuning a poor way to teach a model your company's documentation?
    a: Training adjusts weights toward the style and distribution of the data, not toward a retrievable fact store. The model learns to sound like your docs while still inventing specifics, and every update means retraining.
  - q: What does fine-tuning do better than any amount of prompting?
    a: Consistent form at low cost — a house tone, a rigid output structure, a narrow classification task — and shrinking a task onto a smaller, cheaper model.
deeper:
  - label: "IBM Technology — Is RAG still needed, given long context windows?"
    url: https://www.youtube.com/watch?v=UabBYexBD4k
  - label: "IBM Technology — Is fine-tuning still needed? LLMs, RAG and LoRA"
    url: https://www.youtube.com/watch?v=-W2JdSl1v48
related_formulas: []
---

## Facts against form

Think of it like training a new employee. You hand them a reference manual to
look up policies as needed — that's retrieval. You also coach them on how to
phrase an answer, how formal to be, how to structure a reply — that's
fine-tuning. One supplies facts at the moment they're needed; the other
shapes behavior permanently. If the requirement is "it must cite our current
pricing," that's a fact, it changes on Tuesday, and it belongs in retrieval.
If the requirement is "it must always answer in our support tone, in this
exact structure, without a preamble," that's form — stable over time, and
fine-tuning handles it cheaply and consistently for every call afterward.
"Fine-tune it on our docs" conflates the two, and it's the instinct nearly
every team reaches for first. It fails in a specific, predictable way: the
model absorbs the vocabulary and cadence of the documentation, sounds more
fluent and confident about the domain, and keeps inventing specifics anyway —
just in a more convincing voice, which makes the invented details harder to
catch, not easier.

## A decision checklist

A short set of questions settles most real cases. Does the correct answer
change over time, even occasionally? If yes, that argues strongly for
retrieval, since a fine-tuned fact becomes wrong the moment the underlying
reality changes and only a retrain fixes it. Does the task need a specific,
narrow, repeatable output shape far more than it needs new facts? That argues
for fine-tuning. Is the corpus larger than any context window could hold, or
does it need per-user access restrictions enforced at query time? That is
retrieval's job, since fine-tuning bakes training data into the weights with
no per-request access control at all. Is latency or cost at very high volume
the binding constraint, more than either facts or form? A fine-tuned smaller
model can beat a larger general model on cost at scale for a narrow, stable
task, which is its own separate reason to fine-tune, covered further in the
[fine-tuning](/roadmaps/ai-engineer/fine-tuning) lesson.

## Hybrid designs, which is what most mature systems actually run

These are not mutually exclusive, and treating the choice as all-or-nothing
misses how real production systems are usually built. A common and effective
combination fine-tunes a smaller model purely for consistent output format and
tone, while that same model still receives retrieved facts in its prompt at
call time — fine-tuning solves the form problem once, retrieval solves the
facts problem continuously, and neither technique is asked to do the other's
job.

## When it is neither: the cheap options first

Before reaching for either, exhaust the options that cost hours rather than
weeks: a clearer prompt, better context assembly, a more capable off-the-shelf
model. These resolve a surprising number of cases that initially look like
they need retrieval or fine-tuning, simply because the original approach was
underspecified rather than genuinely lacking facts or form.

## The long-context argument, and where it stops working

As context windows grew into the hundreds of thousands of tokens, a real
argument emerged that retrieval could simply be skipped by putting the whole
document set directly in the prompt every time. For a small, stable corpus —
a single contract, one codebase, a handful of fixed policies — this is
genuinely the simpler design, and it is the right one to take rather than
building a retrieval pipeline for a problem that does not need one. It stops
working for predictable reasons as the corpus grows: cost scales with every
single call rather than with the corpus size once, latency grows with the
input, and accuracy degrades as the window fills — the lost-in-the-middle
effect covered in the
[context window](/roadmaps/ai-engineer/the-context-window) lesson. Retrieval
earns its complexity once the corpus is meaningfully larger than what fits
comfortably in a window, changes often enough that re-sending it all every
time is wasteful, or needs to produce citations pointing at a specific
source rather than an undifferentiated block of text.

The honest summary: these are not really competitors most of the time. Mature
systems retrieve facts, occasionally fine-tune a small model for a narrow
step, and spend the majority of their engineering effort on neither — on the
evaluation, observability and operational discipline covered later in this
roadmap.
