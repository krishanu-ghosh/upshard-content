---
title: Choosing a model
description: Open weights against closed APIs, size against latency and cost, and why a benchmark leaderboard is the weakest evidence available to you.
roadmap: ai-engineer
node: choosing-a-model
group: working-with-models
video:
  provider: youtube
  id: "pYax2rupKEY"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 416
level: intermediate
prerequisites: [the-context-window]
takeaways:
  - The decision is a trade-off between capability, latency, cost and control, and no single model wins all four.
  - Public benchmarks are contaminated, aggregated and rarely resemble your task, so they rank models on something other than your problem.
  - The only evidence that settles the question is your own eval set run against the candidates.
check_questions:
  - q: Why is a leaderboard position weak evidence for your application?
    a: It aggregates performance on public tasks that may be in the training data and almost certainly differ from yours. It tells you a model is generally capable, not that it is right for your problem.
  - q: When does self-hosting an open-weights model actually pay?
    a: When volume is high and steady enough to keep hardware utilised, or when data residency and control are requirements rather than preferences. At low or spiky volume, per-token pricing is cheaper than idle GPUs.
deeper:
  - label: "IBM Technology — LLM & AI Agent Benchmarks vs Reality"
    url: https://www.youtube.com/watch?v=nVImVgKpoOY
  - label: "Manifold AI Learning — API vs self-hosted LLMs, with the cost arithmetic"
    url: https://www.youtube.com/watch?v=-Q6OQmt_BoU
related_formulas: [model-memory-footprint, chinchilla-scaling-law]
---

## Four axes, no winner

Capability, latency, cost and control. Every model sits somewhere on those
four, and pushing on one usually costs you another. A frontier model is the
most capable, and also the slowest and most expensive. A small open-weights
model is cheap and fast, and will fail on the hard tail of your inputs.
There's no dominant choice, only one that fits what you're building — and
most applications aren't one workload to begin with.

Think of it like staffing a team: you don't put your most senior, most
expensive engineer on routing support tickets. A cheap model handles
routing, extraction, classification; an expensive one handles the step that
actually needs judgement. Picking per call — a pattern sometimes called a
**model router** or **cascade** — is usually better engineering than picking
one model for the whole product. Send every request first to a small model,
and only escalate to a larger one when a cheap confidence signal — a low
score, a flagged uncertainty, a failed validation — says the easy model
wasn't enough.

## Benchmarks, and specifically why they mislead

Public leaderboards suffer from problems worth naming individually, not just
gesturing at. **Contamination**: some fraction of a benchmark's test
questions have very likely appeared, verbatim or near enough, in a model's
training data, inflating its apparent score on exactly that benchmark while
telling you nothing about genuinely novel inputs. **Aggregation**: a single
headline score averages across wildly different task types, so two models
can tie on the overall number while one is far better at your specific task
and far worse at everything else the benchmark measured.

Crowd-voted arenas, where people compare two anonymous responses and pick a
favorite, carry their own bias: people tend to reward longer, more
confident-sounding answers regardless of correctness — the same verbosity
bias covered in [LLM as judge](/roadmaps/ai-engineer/llm-as-judge). And
narrow synthetic tests like needle-in-a-haystack retrieval checks measure one
specific failure mode and say nothing about reasoning, tool use, or your
actual document format.

## The cost arithmetic of self-hosting

Running an open-weights model yourself trades a per-token bill for fixed
infrastructure cost, and the trade only pays off at real, steady volume. A
model's parameters occupy GPU memory roughly in proportion to their count and
numeric precision — a seven-billion-parameter model at 16-bit precision
needs roughly fourteen gigabytes just to hold the weights, before the `KV`
cache for whatever conversations are being served concurrently. Quantizing
to 8-bit or 4-bit cuts that memory requirement roughly in half or further, at
some cost to quality (see
[model memory footprint](/formulas/model-memory-footprint)).

Below a threshold of sustained request volume, a GPU sitting partly idle
costs more per answered request than paying a provider per token. Above that
threshold, self-hosting wins decisively, because the hardware cost is fixed
no matter how hard it's worked. The crossover point depends on model size,
hardware pricing, and your actual traffic pattern — which is exactly why it
needs calculating for your situation, not assumed from a rule of thumb.

## Licensing is part of the decision, not an afterthought

"Open weights" doesn't mean one license. Some open-weight models permit
unrestricted commercial use. Others restrict use above a revenue threshold,
forbid using the model's outputs to train a competing model, or carry other
conditions that matter the moment the product generates real revenue. Settle
this before building a dependency on a model, not after.

## Building in the ability to switch

Whatever you choose today won't be what you run in a year — model releases,
pricing, and your own volume all shift faster than most product roadmaps. An
abstraction layer between your application code and the specific provider
API, even a thin one, turns a model change into a configuration update
instead of a rewrite. And an
[eval set](/roadmaps/ai-engineer/building-an-eval-set) run against every
candidate is what answers the choice with evidence instead of a leaderboard
position.
