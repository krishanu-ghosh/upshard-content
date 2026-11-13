---
title: Local and edge inference
description: Running inference on consumer hardware or at the edge rather than a datacenter — far tighter memory and power constraints, and the tradeoffs that forces.
roadmap: inference-engineering
node: local-and-edge-inference
group: hardware-and-bottlenecks
video:
  provider: youtube
  id: "a2muGkT4WD4"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 651
level: advanced
prerequisites: [choosing-hardware]
takeaways:
  - Consumer and edge devices have a tiny fraction of a datacenter GPU's memory and power budget, which forces aggressive quantization and much smaller models to fit at all.
  - Running on-device trades raw capability for properties a datacenter API cannot offer at any price — zero network latency, full data privacy, and operation with no connectivity at all.
  - The same optimization techniques covered elsewhere in this roadmap — quantization, smaller architectures, efficient kernels — matter even more at the edge, where the constraints are tighter by orders of magnitude.
check_questions:
  - q: Why does edge inference typically require far more aggressive quantization than datacenter serving?
    a: A phone or embedded device has a small fraction of a datacenter GPU's memory and power budget, so a model has to be compressed much further just to fit and run within that budget — quantizing to 4-bit or lower is common on the edge, where a datacenter deployment might use 8-bit or 16-bit comfortably.
  - q: What can on-device inference offer that no datacenter API can, regardless of the API's latency?
    a: Operation with no network connection at all, zero network round-trip latency since there is no network hop, and data that never leaves the device at all — properties that matter for privacy, for offline use, and for latency floors a network call can never beat.
deeper:
  - label: "Cormac Brick (Google) — Why Large? Tiny LMs & Agents on Edge/Robotics"
    url: https://www.youtube.com/watch?v=hacEQHHhu2Q
  - label: "NVIDIA, Unsloth, Hugging Face, Ollama — Compression at the Edge"
    url: https://www.youtube.com/watch?v=J4_jCrTxMkk
related_formulas: [model-memory-footprint]
---

## A completely different budget

Everything covered so far in this roadmap assumes datacenter-class hardware
— GPUs with tens or hundreds of gigabytes of memory, abundant power, and a
fast network connection to whoever is calling them. Running inference on a
phone, a laptop, or an embedded device operates under a budget that is
smaller by orders of magnitude on every one of those dimensions, which
changes which techniques from earlier in this roadmap matter most and how
aggressively they have to be applied.

## Why quantization gets pushed further at the edge

[Quantization](/roadmaps/inference-engineering/quantization), covered later
in this roadmap, reduces a model's memory footprint by lowering the numeric
precision its weights are stored at — see
[model memory footprint](/formulas/model-memory-footprint) for the
arithmetic. A datacenter deployment might comfortably use 16-bit or 8-bit
precision, trading a small amount of quality for a meaningful memory saving.
An edge device frequently has no choice but to go further — to 4-bit or
below — simply to fit a useful model into the memory actually available,
accepting a larger quality cost than a datacenter deployment would ever need
to tolerate.

## What you get that no API can offer

Running on-device is not only a compromise forced by weaker hardware — it
buys genuine properties a datacenter API cannot match at any price. There is
no network round trip at all, which sets a latency floor far below what any
remote call could achieve regardless of how fast the server-side inference
is. Data never leaves the device, which matters directly for privacy-
sensitive applications and for regulatory requirements that forbid sending
certain data anywhere at all. And the device keeps working with no network
connection whatsoever, which matters for anything that has to function
reliably offline.

## The specific pressures that make edge hardware different

Beyond simply having less memory, edge devices add constraints a datacenter
GPU never has to consider. **Power** is finite and often battery-backed,
which rules out running hardware anywhere near its peak sustained throughput
for long, in a way a datacenter rack with dedicated cooling and a stable
power supply never has to worry about. **Thermal throttling** is a real,
immediate effect on a phone or a small embedded board in a way it rarely is
in a datacenter — sustained heavy inference can cause the device to slow
itself down to avoid overheating, which shows up as inconsistent, degrading
performance over the course of a long session rather than the steady
throughput a server delivers. And **shared resources** matter in a way they
do not on a dedicated inference server: a phone running inference is also
running everything else the user expects the phone to do at the same time,
competing for the same limited memory and compute.

## Where this leaves hardware strategy

Local and edge inference is not a smaller version of datacenter serving; it
is a different point on the same tradeoff space covered throughout this
group, pushed to its tightest extreme. The same questions —
[what matters about the hardware](/roadmaps/inference-engineering/gpu-architecture-for-inference), whether you are
[memory- or compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound), how to
[size for the workload](/roadmaps/inference-engineering/choosing-hardware) —
all still apply, just against a vastly smaller memory and power budget than
anything else in this roadmap assumes. The deeper links cover this pushed
even further: running genuinely tiny models on robotics-class hardware, and a
panel of practitioners comparing the compression techniques that make any of
this feasible at all.
