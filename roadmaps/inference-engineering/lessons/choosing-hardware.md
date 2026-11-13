---
title: Choosing hardware
description: Sizing and procuring GPUs for a real workload — matching hardware to model size, batch size and latency targets, not just buying the newest chip.
roadmap: inference-engineering
node: choosing-hardware
group: hardware-and-bottlenecks
video:
  provider: youtube
  id: "9tvJ_GYJA-o"
  start: 1241
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 2019
level: advanced
prerequisites: [memory-bound-vs-compute-bound]
takeaways:
  - Sizing hardware starts with memory — a GPU has to hold the model's weights plus enough headroom for the KV cache at your expected concurrency and context length, before anything else matters.
  - Real traffic patterns, not a single assumed request shape, should drive the sizing decision — building input and output length histograms from actual usage reveals what the hardware will really be asked to do.
  - The newest, most expensive GPU is not automatically the right choice; the right one is whichever meets your latency and throughput targets at the lowest cost, which is frequently a smaller, cheaper chip than the headline option.
check_questions:
  - q: Why does sizing hardware start with memory rather than compute throughput?
    a: A GPU that cannot hold the model's weights and the KV cache for your expected concurrency cannot serve the workload at all, regardless of how fast its compute is. Memory capacity is a hard constraint that has to be satisfied before compute throughput becomes relevant.
  - q: Why build an input/output-length histogram from real traffic rather than assuming a typical request shape?
    a: Real traffic is rarely uniform — some requests are short, some carry long context, and the mix determines actual average and peak memory and compute demand. Sizing against an assumed typical request can badly under- or over-provision for what the system actually receives.
deeper:
  - label: "Charles Frye (Modal) — How fast are LLM inference engines anyway?"
    url: https://www.youtube.com/watch?v=DeFF3J8T5Pk
  - label: "Mozhgan Kabiri Chimeh (NVIDIA) — Running LLMs locally on DGX Spark"
    url: https://www.youtube.com/watch?v=c5-kx2bwoCk
related_formulas: [model-memory-footprint, kv-cache-size]
---

## Start with memory, because it is the hard constraint

Before anything about speed or cost, a GPU either has enough memory to hold
a given model and workload or it does not — there is no way to trade around
insufficient memory the way you might trade latency for cost elsewhere. The
starting calculation is always the same shape: how much memory do the
model's weights occupy, per
[model memory footprint](/formulas/model-memory-footprint), plus how much
memory the KV cache will need at your expected concurrency and context
length, per [KV cache size](/formulas/kv-cache-size). Only once a GPU (or a
set of GPUs, if the model needs to be split across several — covered in
[model parallelism](/roadmaps/inference-engineering/model-parallelism)) can
hold both of these does it become a candidate at all.

## Size against real traffic, not an assumed shape

A common mistake is sizing against one assumed "typical" request — a fixed
prompt length, a fixed output length — when real traffic is rarely that
uniform. Building an actual histogram of input and output lengths from
production logs, or from a realistic simulation if nothing is in production
yet, reveals the true distribution: most requests might be short, with a
long tail of much longer ones that disproportionately drive peak memory and
latency. Sizing against the real distribution, including its tail, avoids
both the failure of under-provisioning for peak load and the waste of
over-provisioning for a worst case that rarely actually happens.

## A worked illustration of the sizing arithmetic

Consider a seven-billion-parameter model served at 16-bit precision, which,
per [model memory footprint](/formulas/model-memory-footprint), occupies
roughly fourteen gigabytes just for the weights. Budgeting for a target of
fifty concurrent conversations averaging two thousand tokens of context each
adds a further meaningful chunk of memory for the KV cache on top of that,
per [KV cache size](/formulas/kv-cache-size) — the exact figure depends on
the model's specific architecture, including whether it uses the
memory-saving attention variants covered in
[attention variants for serving](/roadmaps/inference-engineering/attention-variants-for-serving). Add both together, leave genuine headroom rather than
sizing to the exact byte, and the result is the actual memory a candidate GPU
needs to clear before it is even a candidate worth comparing on speed or
cost at all — which is why this arithmetic, however approximate, has to come
first.

## The newest chip is not automatically the right answer

It is tempting to default to whichever GPU generation is newest and most
capable, but the right choice is whichever one meets your latency and
throughput targets at the lowest total cost — which is frequently a smaller,
older, cheaper chip once the actual workload is sized correctly, not the
flagship option. A model that comfortably fits and serves well within target
latency on a mid-tier GPU gains nothing from a more expensive one beyond
unused headroom; that budget is better spent on more of the mid-tier
hardware to handle more concurrent traffic, or simply saved. The deeper links
cover both ends of this decision concretely: a head-to-head benchmark across
engines and model sizes, and a case study showing how much precision format
and memory bandwidth, rather than raw chip tier, actually drive what a single
box can serve.
