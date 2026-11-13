---
title: The inference stack
description: The layers between a trained checkpoint and an API that serves it — hardware, engine, serving framework — and where performance work happens at each.
roadmap: inference-engineering
node: the-inference-stack
group: foundations
video:
  provider: youtube
  id: "woIYJYd_etI"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 3598
level: advanced
prerequisites: [open-vs-closed-models]
takeaways:
  - A model checkpoint on disk is not a running service; several distinct layers sit between it and an API a user can call, and each layer is where a different category of performance work happens.
  - The inference engine handles the model-specific heavy lifting — batching, memory management, the actual forward pass — while the serving framework around it handles HTTP, routing and scheduling across possibly many instances.
  - Treating "make inference faster" as one problem rather than a stack of separable layers is the most common reason performance work stalls — each layer has its own bottlenecks and its own fixes.
check_questions:
  - q: What is the practical difference between an inference engine and a serving framework?
    a: The inference engine is concerned with running the model itself efficiently on the hardware — batching requests, managing the KV cache, executing the forward pass. The serving framework sits around it, handling incoming HTTP requests, routing them to an available instance, and coordinating multiple engine instances together.
  - q: Why does it help to think of "making inference faster" as a stack of layers rather than one problem?
    a: Each layer has entirely different bottlenecks and entirely different fixes — a GPU memory-bandwidth limit is solved differently from a request-routing inefficiency, which is solved differently from a tokenizer being unnecessarily slow. Treating them as one undifferentiated problem makes it hard to know where to actually look.
deeper:
  - label: "Kyle Kranen (NVIDIA) — Hacking the Inference Pareto Frontier"
    url: https://www.youtube.com/watch?v=Y2qc0UhDSnc
related_formulas: []
---

## From a file on disk to an answer for a user

A trained model is, physically, a large file of numbers. Getting from that file to
a working API endpoint that answers a user's request in a reasonable time involves
several genuinely separate layers, and the rest of this roadmap is organised around
exactly these layers, one group at a time.

**Hardware** is the GPU (or GPUs) the model actually runs on, with its own compute
throughput and memory bandwidth — covered in the [next group](/roadmaps/inference-engineering/gpu-architecture-for-inference). **The inference engine** is the
software that takes a request and the model's weights and actually computes an
output — tokenizing the input, managing the KV cache, running the forward pass,
batching multiple requests together — covered in the
[serving software](/roadmaps/inference-engineering/inference-engines) group. **The
serving framework** sits around one or many engine instances, accepting HTTP
requests, routing them to an available instance, and handling what happens when
demand exceeds capacity — covered in the
[production](/roadmaps/inference-engineering/autoscaling-and-cold-starts) group at
the end of this roadmap.

## What actually happens inside the engine layer

Zooming into just the inference engine reveals its own set of distinct
steps, each with its own cost. A request's text first passes through a
**tokenizer**, converting it into the token ids the model actually operates
on — cheap, but not free, and a surprisingly common source of subtle bugs
when a tokenizer mismatches the model it is paired with. The **scheduler**
then decides which requests get processed in this step and how they are
batched together, a decision covered in depth in
[continuous batching](/roadmaps/inference-engineering/continuous-batching).
The model itself executes its forward pass, split into the
[prefill and decode](/roadmaps/inference-engineering/prefill-and-decode)
phases covered in the next group. The scheduler is frequently where real
bottlenecks hide, because a slow or naive scheduling decision can leave a
GPU underutilised even when every other layer is working efficiently.

## Tracing a slow request through the stack

Say a user reports a response that took four seconds when it usually takes
one. The hardware layer might be fine — the GPU is idle half the time — which
rules out a raw compute limit and points at the engine or the framework
above it. Inside the engine, the scheduler might show the request waited in
queue behind a burst of other traffic before its first token was even
computed, which is a batching or admission problem, not a model problem. Or
the engine might have run the request immediately, in which case the delay
has to be in the framework layer — a slow health check, a misconfigured
timeout, an autoscaler that hadn't yet spun up enough capacity. Each of
these has a different fix, and none of them look alike from the outside;
only checking layer by layer, rather than guessing at "the model" or "the
GPU," actually finds which one is responsible.

## Why the layering matters in practice

The most common mistake in performance work on an inference system is treating
"make it faster" as one undifferentiated problem rather than asking which layer is
actually the bottleneck right now. A GPU running at its memory-bandwidth limit needs
a different fix — covered in [memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound) — from a request sitting in a queue
because the serving framework is routing poorly, covered in
[routing and load balancing](/roadmaps/inference-engineering/routing-and-load-balancing). Diagnosing which layer is actually slow, rather than guessing, is most of
the skill this entire roadmap is building toward — and the
[benchmarking](/roadmaps/inference-engineering/benchmarking-inference-systems)
lesson later in this roadmap covers exactly how to find out.
