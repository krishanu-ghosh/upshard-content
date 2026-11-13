---
title: Prompt ensembling
description: Running several prompts or phrasings for the same task and aggregating their answers, so errors that don't correlate across them wash out.
roadmap: prompt-engineering
node: prompt-ensembling
group: output-control
video:
  provider: youtube
  id: "IMkSUxGKfvw"
  start: 0
  end: null
creator:
  name: LearnAwesome
  channel_url: https://www.youtube.com/@LearnAwesome
duration_seconds: 559
level: intermediate
prerequisites: []
takeaways:
  - Self-consistency varies only the random sample from one prompt; prompt ensembling varies the prompt's wording or structure itself, catching errors a single phrasing hides.
  - Errors that are correlated within one prompt, where every sample repeats the same mistake, are often uncorrelated across different prompt phrasings, which is what makes aggregation help.
  - N-way prompt ensembling costs roughly N times as much as a single call, so it's worth reaching for when a wrong answer is expensive, not as a default.
check_questions:
  - q: How does prompt ensembling differ from self-consistency?
    a: Self-consistency samples the same prompt multiple times and takes a majority vote; prompt ensembling runs genuinely different prompt phrasings or structures for the same task and aggregates across those instead.
  - q: Why might a systematic, repeated wrong answer be a sign to use prompt ensembling rather than self-consistency?
    a: If every sample of the same prompt makes the same mistake, the bias lives in the prompt's wording itself, and self-consistency's repeated sampling won't catch it — a genuinely different phrasing is needed to avoid inheriting the same blind spot.
deeper: []
related_formulas: []
---

Self-consistency — sampling the same prompt multiple times and taking the majority
answer — is the best-known instance of a broader idea: running several variations of a
task and combining their outputs tends to be more reliable than running any single one.
Prompt ensembling generalizes that idea past "the same prompt, sampled repeatedly" to
"genuinely different prompts for the same task, each run once or a few times," which
catches a different, complementary category of error.

## Beyond self-consistency: ensembling the prompt itself

Where self-consistency varies only the model's sampling (same instructions, same
examples, different random draws), prompt ensembling varies the prompt itself: three
different phrasings of the same instruction, the same task posed directly versus via a
few-shot example versus via a chain-of-thought request, or the same question asked with
its context presented in a different order. The task is identical; what changes is how
it's described to the model. You then have several independent answers to the same
underlying question, produced by meaningfully different routes rather than different
random seeds down the same route.

## Why diverse phrasings catch different failures

A single prompt's wording can accidentally bias a model toward a particular kind of
mistake — a leading phrase that nudges it toward one answer, an example that anchors it
on an irrelevant pattern, an ambiguous instruction that gets resolved the same, wrong,
way every time it's asked that way. Sampling the same prompt repeatedly, as
self-consistency does, won't catch this: if the prompt itself is the source of the bias,
every sample inherits it. A genuinely different phrasing is far less likely to share the
exact same blind spot, because the bias was a property of that specific wording, not of
the task. This is the core bet behind prompt ensembling: errors that are *correlated*
within one prompt, where every sample makes the same mistake, are often *uncorrelated*
across different prompts, where each phrasing makes different mistakes or none — and
uncorrelated errors are exactly the kind that aggregation washes out.

## Aggregating the results

For tasks with a discrete answer — classification, multiple choice, yes/no — aggregation
is simple majority vote across the different prompts' outputs, the same mechanism
self-consistency uses. For open-ended generation, where there's no single "correct"
string to vote on, aggregation usually means a separate judging step: generate several
candidate responses from the different prompts, then use an additional prompt, or a
human, to pick the best one or synthesize a combined answer. This judging step is itself
a prompt, with its own failure modes, which is worth remembering before treating
ensembling as a free upgrade — it adds a new place for things to go wrong, even as it
removes others.

## The cost you're trading for reliability

Every additional prompt variant is a full additional model call, or several, if you're
also sampling each variant multiple times, so N-way ensembling costs roughly N times as
much in tokens, latency, and money as a single call. That multiplier is the same one
self-consistency pays, and it compounds if you combine the two — three prompt variants
sampled three times each is nine calls for one final answer. This is not a technique to
reach for by default; it's a technique to reach for when a wrong answer is expensive
enough — a misclassified support ticket routed to the wrong team, a factual error that
ships to a user — that paying for redundancy is cheaper than paying for the mistake.

## When it's worth it

Reach for prompt ensembling specifically, rather than plain self-consistency, when you
suspect the failure mode is in how the task is phrased rather than in the model's
reasoning variance: ambiguous instructions, tasks sensitive to example choice, or prompts
where you've noticed the same wrong answer coming back consistently rather than
inconsistently. If errors look random from one sample to the next, self-consistency alone
is usually enough and cheaper. If errors look *systematic* — the same mistake every time
with one phrasing — that's the signal that the prompt itself, not the sampling, is the
variable worth diversifying.
