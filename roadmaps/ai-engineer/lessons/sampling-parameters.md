---
title: Sampling parameters
description: Temperature, top-k, top-p and repetition penalties — what each one does to the distribution, and when touching it is justified.
roadmap: ai-engineer
node: sampling-parameters
group: working-with-models
video:
  provider: youtube
  id: "MkaazQttbpc"
  start: 0
  end: null
creator:
  name: Gary Explains
  channel_url: https://www.youtube.com/@GaryExplains
duration_seconds: 891
level: intermediate
prerequisites: [calling-a-model-api]
takeaways:
  - Sampling happens after the model has finished; these parameters reshape a distribution the model already produced rather than changing how it thinks.
  - Temperature scales the scores before the softmax, flattening the distribution above one and sharpening it below.
  - Top-k and top-p truncate the distribution before sampling, cutting off the long tail of implausible tokens that temperature alone would leave reachable.
check_questions:
  - q: What is the difference between top-k and top-p?
    a: Top-k always keeps a fixed number of candidates. Top-p keeps however many are needed to reach a cumulative probability, so it keeps few when the model is confident and many when it is not.
  - q: Why does raising temperature produce incoherent text rather than creative text?
    a: Temperature flattens the whole distribution, so genuinely wrong tokens gain probability alongside interesting ones. Without a top-p cut the tail stays reachable and eventually one gets sampled.
deeper:
  - label: "DataMListic — LLM sampling: temperature, top-k, top-p on the logits"
    url: https://www.youtube.com/watch?v=-BBulGM6xF0
related_formulas: [softmax, temperature-sampling, top-p-sampling]
---

## Where in the pipeline this happens

The model's job finishes with a score — a **logit** — for every token in the
vocabulary. Every parameter in this lesson operates afterward, on those
scores, deciding which token actually gets picked. Think of it like a judge
who has already scored every contestant: these settings don't change the
scores, only the rule for picking a winner from them. Nothing here makes the
model smarter or dumber — it changes how adventurously you read an answer the
model already computed.

## Temperature, with the arithmetic made concrete

Before the softmax converts logits into probabilities, temperature divides
every logit by a value `T`. At `T = 1` nothing changes. As `T` falls below
one, the gaps between logits are stretched — the already-likely token becomes
even more dominant — and at `T = 0` the distribution collapses entirely onto
the single highest-scoring token every time, which is why temperature zero is
called **greedy decoding** and is fully deterministic (aside from floating
point effects described below). As `T` rises above one, the gaps shrink, and
the distribution flattens toward uniform, so tokens that were only slightly
less likely become nearly as likely as the top choice. A toy example makes
this concrete: suppose three candidate tokens have raw probabilities of 70%,
20% and 10% at T=1. Lowering temperature to 0.5 might push these to roughly
88%, 9% and 3% — the leader pulls further ahead. Raising it to 2.0 might push
them toward 45%, 32% and 23% — the gap nearly closes, and the model becomes
willing to pick what used to be a clear runner-up.

## Top-k and top-p, and why they exist alongside temperature

Temperature alone never removes a token from consideration, no matter how
unlikely — it only reshapes the probabilities, so at a high enough temperature
even a wildly implausible token remains reachable, which is the direct cause
of incoherent output. **Top-k** fixes a hard cutoff: only the k
highest-probability tokens are considered at all, and everything else is
zeroed out before sampling. **Top-p** (nucleus sampling) instead keeps
whichever smallest set of top tokens together accounts for p of the total
probability mass — so when the model is very confident, that set might be just
one or two tokens; when it is uncertain, the set naturally grows to include
more candidates. This adaptiveness is why top-p is generally preferred over a
fixed top-k: it tightens automatically exactly when the model already knows
the answer, and loosens automatically when it does not.

## Repetition and frequency penalties

Separately from the shape of the distribution, a penalty can be subtracted
from the logit of any token based on how many times it has already appeared
in the output so far. A **frequency penalty** scales with how often the token
has recurred; a **presence penalty** applies a flat penalty the moment a token
has appeared at all, regardless of how many times. These are blunt
instruments reached for when a model gets stuck in a repetitive loop, and they
carry a real cost: text that is legitimately supposed to repeat — a bullet
list with a shared prefix, a block of code, a table — gets actively damaged by
them, so they belong low on the list of things to adjust.

## Determinism is not guaranteed even at temperature zero

A subtlety worth knowing: floating-point addition is not perfectly
associative, and a GPU serving many requests in parallel batches can compute
a given request's logits in a slightly different numerical order depending on
what else is being processed alongside it at that moment. The result is that
even temperature-zero output can occasionally differ by a token or two
between two calls with identical input — not because sampling introduced
randomness, but because the underlying arithmetic was not bit-for-bit
identical. If a workflow genuinely requires reproducibility, most providers
expose a `seed` parameter, which improves — but by itself does not
perfectly guarantee — repeatability.

## Sensible defaults

For extraction, classification, routing and anything you will parse
programmatically: low or zero temperature, and leave top-p at its default.
For drafting, brainstorming and conversational replies: moderate default
temperature with a top-p cut around 0.9 to 0.95, so the implausible tail
stays closed while the plausible middle stays fully open. Treat a change to
any of these as a change to your product's behaviour, belonging in the same
eval harness as a prompt change — not a config value set once and forgotten.
