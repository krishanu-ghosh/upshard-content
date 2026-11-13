---
title: Temperature and randomness
description: Temperature reshapes a model's output distribution before sampling — the single knob that moves output from deterministic to wildly varied.
roadmap: prompt-engineering
node: temperature-and-randomness
group: model-fundamentals
video:
  provider: youtube
  id: "IWSutLJbURU"
  start: 0
  end: null
creator:
  name: KodeKloud
  channel_url: https://www.youtube.com/@KodeKloud
duration_seconds: 308
level: beginner
prerequisites: [sampling-parameters]
takeaways:
  - Temperature divides a model's logits before the softmax step, sharpening the output distribution at low values and flattening it at high values.
  - A temperature near 0 is the right default for factual, extraction, and code-generation tasks; higher values suit brainstorming and creative writing.
  - Even a temperature of 0 does not guarantee perfectly identical output across repeated calls on every provider, because other sources of nondeterminism exist in how requests are batched and executed.
check_questions:
  - q: What does temperature actually change about the model's computation?
    a: It scales the logits (the model's raw, pre-probability scores for each possible next token) by dividing them by the temperature value before the softmax function converts them into probabilities. Dividing by a number less than 1 spreads the logits further apart, sharpening the resulting distribution; dividing by a number greater than 1 compresses them, flattening it.
  - q: You are generating a short product description and want it to sound similarly polished across many products, but not word-for-word identical each time. What temperature range fits, and why not 0?
    a: A low-to-moderate value, roughly 0.3–0.6, fits best. Temperature 0 would make the model's output nearly deterministic for the same prompt, which is wrong when the goal is natural, slightly varied phrasing across runs rather than one fixed answer repeated verbatim.
deeper:
  - label: "3Blue1Brown — Temperature in LLMs"
    url: https://www.youtube.com/watch?v=XsLK3tPy9SI
related_formulas: [temperature-sampling, softmax]
---

## What temperature actually does, mechanically

Before a language model samples a token, it has already computed one logit
per vocabulary token — a raw, unbounded real number scoring how plausible
each candidate is. Those logits get converted into a probability
distribution by the [softmax](/formulas/softmax) function. Temperature
intervenes one step earlier: every logit is divided by the temperature value
before softmax runs (see [temperature sampling](/formulas/temperature-sampling)).
Dividing by a number smaller than 1 makes the logits' differences larger in
relative terms, which softmax then turns into a sharper, more
concentrated probability distribution — the model's favorite tokens become
even more dominant. Dividing by a number greater than 1 shrinks the relative
differences between logits, which softmax turns into a flatter distribution
where many tokens have a genuinely competitive chance of being picked.
Temperature does not add randomness directly; it reshapes how much the
*existing* randomness in sampling gets to matter.

## The two extremes, and why neither is usually right

Temperature 0 is a special case: dividing by (or approaching) zero pushes
the distribution toward a single spike on the highest-logit token, making
generation close to deterministic — effectively greedy decoding, always
picking the model's top choice. This sounds ideal for correctness, but
pushed to its limit it inherits greedy decoding's own failure mode: text
that is locally sensible but can loop or read as oddly generic, because the
model never considers its second-best option even when that option was only
a hair behind the first. At the other extreme, a very high temperature
flattens the distribution so much that near-random tokens get a real chance
of being selected, which quickly produces incoherent, broken text — the
model stops favoring words that make grammatical or factual sense. Nearly
every real use case lives between these extremes, and the right setting is
a judgment about how much you value consistency versus variety for the
specific task in front of you.

## Choosing a value for your task

As a starting point: tasks with one correct or clearly best answer —
extracting a field from a document, classifying text, generating code that
needs to run — want temperature near 0, typically 0 to 0.3, because any
deviation from the model's most confident guess is more likely to introduce
an error than a useful alternative. General-purpose assistant chat usually
sits around 0.7, balancing natural-sounding variation with staying on
topic and factually grounded. Creative tasks — brainstorming, fiction,
marketing copy where you want several distinct options to choose from —
benefit from pushing higher, often 0.9 to 1.2 depending on the provider's
scale, accepting a higher rate of odd or incoherent outputs in exchange for
more genuinely different ideas. These are starting points, not laws: the
only way to know the right value for a specific prompt and model is to try
a few settings against real examples of the task and see which output
distribution you actually prefer.

## Temperature 0 is not a determinism guarantee

It's a common assumption that temperature 0 means "the same input always
produces the exact same output," and it's not quite true in practice.
Providers batch multiple requests together for efficiency, and
floating-point arithmetic is not perfectly associative — the order
operations happen to be grouped in during a particular batch can shift
results by a tiny amount, occasionally enough to flip which token has the
(now very sharply favored) highest logit. Some providers also apply
additional sampling-adjacent steps even at temperature 0. The practical
implication: if your system depends on byte-for-byte reproducible output —
for testing, for caching, for auditing — verify that empirically against
the specific provider and model you're using rather than assuming
temperature 0 delivers it by definition, and consider that true
determinism may require a provider-specific seed parameter alongside
temperature, where one is offered.

## How temperature interacts with top-k and top-p

Temperature does not operate alone. It is applied first, reshaping the full
distribution over the entire vocabulary; then top-k or top-p (covered next,
in [top-k and top-p](/roadmaps/prompt-engineering/top-k-and-top-p)) truncates
that reshaped distribution down to a smaller candidate set before the actual
random draw happens. A high temperature with a tight top-p, for instance,
flattens the distribution but then only samples from a small, still-plausible
slice of it — a common way to get varied-sounding but still coherent output.
Treating temperature as the only lever and leaving top-k or top-p at their
defaults is one of the most common prompting mistakes: the two interact, and
tuning one while ignoring the other means you are only ever exploring half
the available control.
