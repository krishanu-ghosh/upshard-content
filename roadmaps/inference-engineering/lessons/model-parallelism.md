---
title: Model parallelism
description: Splitting a model too large for one GPU across several — tensor, pipeline and expert parallelism — and the different tradeoffs each one makes.
roadmap: inference-engineering
node: model-parallelism
group: scaling-and-beyond-llms
video:
  provider: youtube
  id: "DdmgD5RMe1U"
  start: 0
  end: null
creator:
  name: Faradawn Yang
  channel_url: https://www.youtube.com/@faradawnyang6220
duration_seconds: 1218
level: advanced
prerequisites: [choosing-hardware, mixture-of-experts]
takeaways:
  - Tensor parallelism splits individual layers' matrix operations across several GPUs, requiring fast, frequent communication between them since every layer needs every GPU's partial result before continuing.
  - Pipeline parallelism splits different layers across different GPUs instead, passing activations from one stage to the next, which communicates less often but introduces idle time while later stages wait for earlier ones to finish.
  - Expert parallelism spreads a mixture-of-experts model's experts across GPUs, so no single device has to hold every expert in memory, while routing each token's computation to whichever device actually holds the expert it needs.
check_questions:
  - q: Why does tensor parallelism require much more frequent GPU-to-GPU communication than pipeline parallelism?
    a: In tensor parallelism, a single layer's computation is split across GPUs, so every GPU's partial result has to be combined before the next layer can even start — communication happens within every single layer. Pipeline parallelism only passes data between stages at the boundary between layers assigned to different GPUs, which happens far less often.
  - q: What specific problem does expert parallelism solve for mixture-of-experts models that tensor or pipeline parallelism does not address as directly?
    a: An MoE model's full set of experts, as covered in the mixture-of-experts lesson, has to be held in memory even though only a few are active per token. Expert parallelism spreads the experts themselves across GPUs, so the model's enormous total parameter count can be distributed by expert rather than requiring one device to hold them all.
deeper:
  - label: "Tandemn Labs — How Massive LLMs Actually Fit on GPUs (Tensor Parallelism)"
    url: https://www.youtube.com/watch?v=3-gmM450e4E
related_formulas: [model-memory-footprint]
---

## When one GPU is not enough

[Choosing hardware](/roadmaps/inference-engineering/choosing-hardware)
covered sizing a model to fit on a GPU, per
[model memory footprint](/formulas/model-memory-footprint) — but some models
are simply too large for any single GPU's memory, however well chosen. Model
parallelism splits such a model across several GPUs working together, and
the different ways to split it make genuinely different tradeoffs, each
suited to different situations.

## Tensor parallelism: splitting inside a layer

**Tensor parallelism** splits the matrix operations within individual layers
across multiple GPUs — each GPU holds a slice of a layer's weights and
computes its own partial result, which then has to be combined with every
other GPU's partial result before the layer's output is complete and the
next layer can begin. This requires fast, frequent communication between the
GPUs involved, since this combination happens within every single layer,
which is why tensor parallelism is usually confined to GPUs connected by a
very high-bandwidth link within the same machine, rather than spread across
separate machines connected by a slower network.

## Pipeline parallelism: splitting across layers

**Pipeline parallelism** instead assigns entire different layers to
different GPUs — the first several layers run on one GPU, the next several
on another, and so on — with activations passed from one stage to the next
as data flows through the full set of layers. This communicates far less
often than tensor parallelism, since data only crosses between GPUs at the
boundary between stages rather than within every layer, which makes it
practical across GPUs connected by a slower network, even across separate
machines. The cost is idle time: a later stage has to wait for an earlier
one to finish before it has anything to work on, which can leave some GPUs
briefly idle while others are still processing.

## Expert parallelism: splitting by expert

For a [mixture-of-experts](/roadmaps/inference-engineering/mixture-of-experts) model specifically, **expert parallelism** spreads the model's
individual experts across different GPUs, so that no single device has to
hold every expert in memory at once. Each token's computation is routed, at
runtime, to whichever device actually holds the expert the router selected
for it — which means this form of parallelism has to handle genuinely
dynamic, per-token communication patterns, since which GPU a given token
needs depends on the router's decision for that specific token, not on a
fixed, predetermined assignment the way tensor or pipeline parallelism's
communication patterns are.

## Combining them in practice

Real large-scale serving deployments frequently combine more than one of
these simultaneously — tensor parallelism within a high-bandwidth group of
GPUs in one machine, pipeline parallelism across several such machines, and
expert parallelism layered on top for an MoE model — rather than treating
them as mutually exclusive choices. Which combination makes sense depends on
the specific model's architecture, its total size, and the interconnect
available between the GPUs it will run on, which is exactly the kind of
sizing decision
[disaggregated serving](/roadmaps/inference-engineering/disaggregated-serving), the next lesson, extends even further by separating prefill and
decode onto entirely different hardware pools.
