---
title: GPU architecture for inference
description: What actually matters about a GPU for serving — compute throughput, memory bandwidth, memory capacity — and how hardware generations change these numbers.
roadmap: inference-engineering
node: gpu-architecture-for-inference
group: hardware-and-bottlenecks
video:
  provider: youtube
  id: "y-UGrYbJsJk"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1200
level: advanced
prerequisites: [the-inference-stack]
takeaways:
  - A GPU for serving is characterised by three numbers that matter more than almost anything else — compute throughput (FLOPs), memory bandwidth, and memory capacity — and which one is the bottleneck depends on the workload.
  - Tensor cores accelerate the specific low-precision matrix multiplication that dominates both training and inference, and a GPU's generation determines how fast and at what precision those tensor cores run.
  - A newer GPU generation typically improves memory bandwidth and low-precision compute together, which is exactly the combination that benefits LLM inference, since decode is usually memory-bandwidth-bound.
check_questions:
  - q: What are the three numbers that matter most about a GPU for serving?
    a: Compute throughput (how many floating-point operations it can perform per second), memory bandwidth (how fast it can move data between memory and compute units), and memory capacity (how much data — weights and KV cache — it can hold at once).
  - q: Why does a newer GPU generation's improved memory bandwidth matter so much for LLM inference specifically?
    a: Decode, the token-by-token generation phase, is typically memory-bandwidth-bound rather than compute-bound, so the speed at which weights can be read from memory — not raw compute throughput — is usually the limiting factor for how fast tokens are generated.
deeper:
  - label: "Dylan Patel (SemiAnalysis) — System Design for Next-Gen Frontier Models"
    url: https://www.youtube.com/watch?v=gFyBdBm0AGo
related_formulas: [arithmetic-intensity, model-memory-footprint]
---

## Three numbers, and which one bites depends on the workload

A GPU's suitability for serving comes down to three characteristics.
**Compute throughput** measures how many floating-point operations it can
perform per second — its raw arithmetic speed. **Memory bandwidth** measures
how fast data can move between the GPU's memory and its compute units — how
quickly it can actually feed its arithmetic units with the numbers they need.
**Memory capacity** measures how much data — a model's weights, plus the KV
cache for every concurrent request — it can hold in memory simultaneously.
Different workloads are limited by different ones of these three: as covered
in
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound), prefill tends to be limited by compute throughput and
decode by memory bandwidth, while serving many long conversations at once is
often limited simply by memory capacity.

## Tensor cores, and why precision matters

Modern GPUs include dedicated **tensor cores** — hardware specifically built
to accelerate the matrix multiplication that dominates both training and
inference, at reduced numerical precision (16-bit, 8-bit, or lower rather
than the 32-bit precision ordinary arithmetic might use). Running at lower
precision trades a small amount of numerical accuracy for a substantial
speed gain, which is why
[quantization](/roadmaps/inference-engineering/quantization), covered later
in this roadmap, is as effective as it is — it deliberately targets exactly
the precision these specialised cores are fastest at.

## Why hardware generations matter to this roadmap

Each new GPU generation typically improves memory bandwidth and
low-precision compute throughput together, and that specific combination is
exactly what benefits LLM inference the most, given how often decode is the
memory-bandwidth-bound phase. This is why the specific hardware generation a
team is serving on genuinely changes the right answers to questions covered
elsewhere in this roadmap — what batch size is sustainable, what quantization
level is worth using, how many concurrent users one GPU can serve — and why
any specific numbers quoted in this field age quickly as new generations
arrive. The concepts in this lesson outlast any specific chip; the exact
numbers do not.

## Why these three numbers rarely improve at the same rate

It is tempting to think of a GPU generation as uniformly "better" than the
last, but compute throughput, memory bandwidth and memory capacity typically
improve at different rates from one generation to the next, and not always
in proportion to each other. A generation can deliver a large jump in raw
compute while improving memory bandwidth only modestly, which actually
narrows the advantage for a memory-bound workload even as the chip's
headline specifications look dramatically better. This is exactly why
reading a GPU's marketing specifications in isolation can mislead: the
number that matters for your specific workload might be the one that barely
moved, while the one that moved the most is largely irrelevant to what you
are actually running.

## Where this feeds into the rest of the roadmap

Understanding these three characteristics is the foundation for
[the roofline model](/roadmaps/inference-engineering/arithmetic-intensity-and-the-roofline-model) in the next lesson, which formalises exactly how compute
throughput and memory bandwidth together predict whether a given operation
will run at its compute limit or its memory limit — and for
[choosing hardware](/roadmaps/inference-engineering/choosing-hardware) later
in this group, which is this lesson's three numbers applied to an actual
sizing decision.
