---
title: Mixture of experts
description: Sparse MoE models route each token to a subset of expert sub-networks, changing the compute/memory tradeoff versus a dense model of similar size.
roadmap: inference-engineering
node: mixture-of-experts
group: inside-the-forward-pass
video:
  provider: youtube
  id: "tzRvcTEapzo"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 3195
level: advanced
prerequisites: [attention-variants-for-serving]
takeaways:
  - A mixture-of-experts model routes each token to a small subset of specialised sub-networks (experts) rather than passing every token through all of the model's parameters.
  - This decouples total parameter count from the compute cost of processing one token — a model can have far more total parameters than a comparable dense model while running each token through only a fraction of them.
  - Every expert still has to be held in GPU memory even though only a few are active per token, which means MoE models shift the serving tradeoff toward needing more memory but less compute per token than a dense model.
check_questions:
  - q: Why does a mixture-of-experts model let you have more total parameters without a proportional increase in compute cost per token?
    a: A router selects only a small subset of experts to actually process each token, rather than passing every token through every parameter. Total parameter count can grow by adding more experts, while the compute cost per token stays tied to however many experts are actually activated, not the full total.
  - q: What does a mixture-of-experts model still need from memory, even though it uses little compute per token?
    a: Every expert has to be resident in GPU memory at all times, because any token might be routed to any of them — there is no way to predict in advance which experts a given request will need. This means memory requirements scale with total parameter count even though compute does not.
deeper:
  - label: "Trelis Research — Understanding Mixture of Experts"
    url: https://www.youtube.com/watch?v=0U_65fLoTq0
related_formulas: []
---

## Decoupling size from cost

Every model covered so far in this roadmap is **dense**: every parameter
participates in processing every token. A **mixture-of-experts (MoE)** model
breaks this coupling deliberately. Instead of one set of parameters that
every token passes through, the model is organised into many smaller
sub-networks called **experts**, and a learned **router** decides, for each
individual token, which small subset of experts should actually process it
— commonly just one or two out of dozens or more available.

## Why this changes the serving tradeoff

In a dense model, doubling the parameter count roughly doubles the compute
required per token, because every added parameter sits on the path every
token travels. In an MoE model, adding more experts increases the model's
total parameter count without increasing the compute cost per token at all,
as long as the number of experts activated per token stays fixed. This is
the entire appeal: a much larger total model, with much more capacity spread
across its experts, running at close to the compute cost of a far smaller
dense model.

## The memory bill MoE does not avoid

The saving MoE buys is in compute, not memory. Every expert has to sit
resident in GPU memory at all times, because the router's decision happens
per token, at runtime, and there is no way to know in advance which experts
any given request will need — predicting that and loading experts on demand
would introduce its own serving complexity and latency cost. This means an
MoE model's memory footprint scales with its full parameter count, including
every expert whether or not it happens to be used by the token currently
being processed, even though its compute cost scales only with however many
experts are actually activated.

## Why the router itself is worth paying attention to

The router is a small learned component, separate from the experts
themselves, that decides which experts should handle a given token. Its
quality directly determines whether an MoE model's capacity is actually used
well: a router that routes tokens evenly and sensibly across experts lets the
model's full capacity contribute meaningfully, while a poorly trained router
that collapses onto favouring only a handful of experts effectively wastes
most of the model's extra parameters, since the unused experts contribute
nothing to the tokens that never reach them. This routing decision also has a
direct serving-time consequence: an unevenly loaded set of experts means some
GPUs handling popular experts in an [expert-parallel](/roadmaps/inference-engineering/model-parallelism) deployment sit under heavier load than others
handling rarely-selected ones, which is its own load-balancing problem layered
on top of everything else this roadmap covers about distributing work evenly
across hardware.

## What this means for serving hardware and scale

This memory-heavy, compute-light profile is exactly why MoE models are a
natural fit for the
[model parallelism](/roadmaps/inference-engineering/model-parallelism)
techniques covered later in this roadmap, and specifically for **expert
parallelism** — spreading different experts across different GPUs, so that
no single device has to hold every expert in memory at once, while routing
each token's computation to whichever device actually holds the expert it
needs. An MoE model that would be far too large in total parameters to fit
on one GPU can become practical to serve once its experts are distributed
this way, which is a serving-architecture decision these models force in a
way dense models never do.
