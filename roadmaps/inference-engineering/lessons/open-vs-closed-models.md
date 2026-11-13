---
title: Open vs closed models
description: Calling a closed model's API versus self-hosting an open-weight one — and how that choice changes what you actually control.
roadmap: inference-engineering
node: open-vs-closed-models
group: foundations
video:
  provider: youtube
  id: "ePMvfa8vgL8"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1134
level: advanced
prerequisites: [what-is-inference-engineering]
takeaways:
  - Calling a closed model's API means someone else has already done all the inference engineering — fast to start, but you control neither cost, latency, nor the model's behaviour beyond what the API exposes.
  - Self-hosting an open-weight model trades that convenience for full control over hardware, serving stack and cost structure, at the price of now owning the entire operational burden this roadmap covers.
  - The decision is rarely permanent or all-or-nothing — many products call a closed API for most traffic and self-host for the volume, latency, or data-residency needs the API cannot meet.
check_questions:
  - q: What do you give up by calling a closed model's API instead of self-hosting?
    a: Control over cost (you pay the provider's price per token, with no ability to tune the serving stack underneath it), control over latency (bounded by what the provider's infrastructure delivers), and the ability to run the model on your own hardware for data-residency or compliance reasons.
  - q: Why might a team self-host even though it costs more engineering effort than calling an API?
    a: At high enough sustained volume, owning the hardware is cheaper than paying per token indefinitely. It may also be the only way to meet a latency target the API cannot guarantee, or a data-residency requirement that forbids sending data to a third party at all.
deeper:
  - label: "AI Engineer — Workshop: AI Engineering 201, Inference"
    url: https://www.youtube.com/watch?v=N7lJY5IKVLE
related_formulas: []
---

## Someone else's inference engineering, or your own

Calling a closed model through an API means every lesson in this roadmap has
already been handled by someone else — the provider chose the hardware, built the
serving stack, tuned the batching, and absorbed the operational burden of keeping it
running. You get a simple interface and none of the control: you cannot change the
hardware, cannot tune the serving engine, and pay whatever the provider's pricing
says per token.

Self-hosting an open-weight model inverts this completely. You choose the GPUs, pick
or build the serving engine, own the scaling and the on-call rotation — and in
exchange you control every lever this roadmap covers: cost per request, the exact
latency profile, data residency, and the freedom to fine-tune or modify the model
itself.

## The decision is rarely all-or-nothing

In practice, most real products do not pick one side permanently. A team might call
a closed API for most traffic, where the convenience outweighs the cost, and
self-host a smaller open-weight model for one high-volume, latency-sensitive path
where the economics of self-hosting clearly win once volume is high enough. A
company handling regulated data might be required to self-host regardless of cost,
because sending that data to a third-party API is simply not permitted. The right
split is an economic and operational calculation specific to the product, not a
one-time philosophical choice.

## A rough rule of thumb for the crossover

Where volume is genuinely the deciding factor, a useful way to think about it
is comparing the per-token price a closed API charges against the fully
loaded cost of a GPU — purchase or rental price amortised over its useful
life, plus power and the engineering time to operate it — divided by how many
tokens that GPU can actually serve over the same period. At low volume, the
GPU sits partly idle between requests and loses this comparison badly,
since you are paying for capacity you are not using. At high, sustained
volume, the GPU's fixed cost gets spread across enough tokens that it wins
decisively, often by a wide margin, because the API's per-token price has to
include the provider's own margin and their own amortised infrastructure
cost on top of the same underlying hardware economics. Where exactly this
crossover sits depends on the model size, the hardware chosen, and how close
to constantly utilised the self-hosted capacity actually runs — which is why
it is worth calculating for your own real traffic rather than assumed from a
rule someone else quotes.

## What this roadmap assumes from here

Every lesson that follows — hardware, serving engines, quantisation, batching,
scaling across GPUs — matters only once you have chosen, for at least part of your
traffic, to run inference yourself. If you call a closed API exclusively, most of
this roadmap describes work someone else is already doing on your behalf. The value
in understanding it anyway is knowing what that provider is actually doing for the
price you pay, and recognising the exact moment, if it ever comes, when owning that
work yourself would serve your product better.
