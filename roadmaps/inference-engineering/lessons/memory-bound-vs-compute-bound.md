---
title: Memory-bound vs compute-bound
description: Why LLM decode is usually memory-bandwidth-bound rather than compute-bound, and what that implies about where optimization effort actually pays off.
roadmap: inference-engineering
node: memory-bound-vs-compute-bound
group: hardware-and-bottlenecks
video:
  provider: youtube
  id: "4b6KpFN7Rio"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 740
level: advanced
prerequisites: [arithmetic-intensity-and-the-roofline-model, kv-cache-mechanics]
takeaways:
  - Decode moves an entire model's weights through memory to produce just one new token per request, which is too little arithmetic relative to that data movement to keep a GPU's compute units busy.
  - Being memory-bound means more raw compute throughput does not speed decode up at all — only faster memory bandwidth, or doing less data movement per useful token, actually helps.
  - This single fact — decode is usually memory-bound — is the underlying reason behind batching, quantization and several other optimizations covered elsewhere in this roadmap.
check_questions:
  - q: Why doesn't a GPU with more raw compute throughput make decode meaningfully faster?
    a: Decode's bottleneck is how fast the model's weights can be read from memory, not how much arithmetic the GPU can perform. A faster compute unit sitting idle while waiting on memory reads gains nothing from being faster — the wait is the actual limiting factor.
  - q: What does the "memory wall" refer to, and why does it specifically affect LLM inference?
    a: It refers to the growing gap between how fast compute throughput has improved on modern chips versus how much more slowly memory bandwidth has improved. LLM decode depends heavily on memory bandwidth rather than compute, so it suffers more from this gap than compute-bound workloads do.
deeper: []
related_formulas: [memory-bound-token-latency, arithmetic-intensity]
---

## The question this lesson answers plainly

Applying the [roofline model](/roadmaps/inference-engineering/arithmetic-intensity-and-the-roofline-model) from the previous lesson to LLM decode specifically gives a clear, practically important answer: decode is almost always
memory-bound, not compute-bound. This single fact shapes an enormous amount of
what inference engineering actually does, which is why it gets its own
lesson rather than being left as an abstract consequence of the roofline
framework.

## Why decode specifically is memory-bound

Each decode step generates exactly one new token per request. To do that, the
GPU has to read the model's entire set of weights from memory — every layer,
every parameter the forward pass touches — but performs comparatively little
arithmetic with them, since there is only one new token's worth of
computation happening. See
[memory-bound token latency](/formulas/memory-bound-token-latency) for the
arithmetic behind exactly how this time is spent. The ratio of computation to
data moved is low, which, per
[arithmetic intensity](/formulas/arithmetic-intensity), places decode under
the memory roof on essentially every current GPU: the GPU's compute units sit
comparatively idle, waiting for data rather than being kept busy by it.

## What being memory-bound actually implies

The practical consequence is blunt and easy to miss: if decode is
memory-bound, making the GPU's raw compute faster does nothing for decode
speed at all — the bottleneck was never compute in the first place. What
genuinely helps is either increasing memory bandwidth (a hardware choice,
covered in
[GPU architecture](/roadmaps/inference-engineering/gpu-architecture-for-inference)) or reducing how much data has to move per useful unit of output.

## The memory wall, as a trend rather than a one-time fact

This gap is not static — it has been widening over successive hardware
generations. Compute throughput on modern accelerators has grown
dramatically faster than memory bandwidth has over the same period, a trend
often called the **memory wall**: chips keep getting better at arithmetic
far faster than they get better at moving data to feed that arithmetic. For
a compute-bound workload this trend is pure upside, since compute is the
resource getting cheaper fastest. For a memory-bound workload like decode,
it means the bottleneck keeps getting relatively worse generation over
generation, even as the chips themselves keep improving in absolute terms —
which is a large part of why so much of the optimisation work covered later
in this roadmap specifically targets reducing memory traffic rather than
assuming faster hardware alone will solve the problem.

## Where this fact drives the rest of this roadmap

Several techniques covered later in this roadmap exist specifically because
decode is memory-bound. **Batching multiple requests together**, covered in
[continuous batching](/roadmaps/inference-engineering/continuous-batching),
amortises the cost of reading the weights from memory across several
requests' worth of useful computation at once, which directly improves the
ratio a memory-bound operation cares about.
**[Quantization](/roadmaps/inference-engineering/quantization)** reduces the
number of bytes the weights occupy, so the same memory bandwidth moves more
useful information per second — a direct attack on a memory-bound
bottleneck. Recognising which of these levers actually addresses your
bottleneck, rather than reaching for whichever optimisation is most familiar,
starts with knowing whether you are memory-bound or compute-bound in the
first place, which is exactly what this lesson and the previous one together
let you determine.
