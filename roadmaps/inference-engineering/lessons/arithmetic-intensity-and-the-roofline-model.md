---
title: Arithmetic intensity and the roofline model
description: The ratio of compute to memory movement an operation needs, and the roofline model that predicts whether it is compute- or memory-bound on given hardware.
roadmap: inference-engineering
node: arithmetic-intensity-and-the-roofline-model
group: hardware-and-bottlenecks
video:
  provider: youtube
  id: "7EJjdDLK4cg"
  start: 0
  end: null
creator:
  name: Faradawn Yang
  channel_url: https://www.youtube.com/@faradawnyang6220
duration_seconds: 1413
level: advanced
prerequisites: [gpu-architecture-for-inference]
takeaways:
  - Arithmetic intensity is the ratio of floating-point operations performed to bytes of data moved from memory — a measure of how much useful work an operation extracts from each byte it reads.
  - The roofline model plots achievable performance against arithmetic intensity, with a sloped "memory roof" and a flat "compute roof" meeting at a ridge point specific to each GPU.
  - An operation below the ridge point is memory-bound no matter how fast the GPU's raw compute is; an operation above it is compute-bound no matter how much memory bandwidth is available.
check_questions:
  - q: What does a low arithmetic intensity actually mean about an operation?
    a: It means the operation performs relatively little computation for each byte of data it reads from memory — most of its time is spent waiting on data movement rather than doing arithmetic, which is the definition of being memory-bound.
  - q: What is the ridge point in a roofline chart, and why does it depend on the specific GPU?
    a: It is the arithmetic intensity at which an operation transitions from memory-bound to compute-bound, found where the sloped memory roof meets the flat compute roof. Its exact position depends on the specific GPU's memory bandwidth and peak compute throughput, so the same operation can be memory-bound on one GPU and compute-bound on another.
deeper: []
related_formulas: [arithmetic-intensity]
---

## A ratio that predicts the bottleneck before you measure anything

[Arithmetic intensity](/formulas/arithmetic-intensity) is the ratio of
floating-point operations performed to bytes of data moved from memory for a
given operation. A high arithmetic intensity means an operation does a lot of
computation per byte it reads — it keeps the GPU's compute units busy
relative to how much it depends on memory movement. A low arithmetic
intensity means the opposite: comparatively little computation happens per
byte moved, and the operation spends more of its time waiting on memory than
computing.

## The roofline model, built from two lines

The **roofline model** plots this relationship for a specific piece of
hardware. One line — the **memory roof** — rises with arithmetic intensity,
representing the maximum performance achievable given the GPU's memory
bandwidth: at low arithmetic intensity, performance is capped by how fast
data can be moved, regardless of how much raw compute the GPU has. A second,
flat line — the **compute roof** — represents the GPU's peak possible
throughput, which no operation can exceed no matter how favourable its
arithmetic intensity. These two lines meet at a single point for any given
GPU, called the **ridge point**.

## A worked comparison: two matrix multiplies

Picture two operations on the same GPU. The first multiplies a `4096x4096`
weight matrix by a single `4096`-element vector — one decode step for one
token. The second multiplies the same weight matrix by a batch of 256 such
vectors stacked together. Both touch the same weight matrix, so both move
roughly the same number of bytes from memory. But the batched version performs
256 times more arithmetic for that same memory traffic, so its arithmetic
intensity is roughly 256 times higher. On a roofline chart the single-vector
multiply sits far to the left, deep under the memory roof; the batched version
sits much further right, potentially past the ridge point and under the
compute roof instead. Same weights, same GPU, same kind of operation — wildly
different position on the chart, purely because batching changed how much
compute rides along with each byte read.

## Reading an operation's position on the chart

An operation's arithmetic intensity determines where it sits on this chart,
and that position tells you immediately which of the two roofs actually
bounds it. An operation with arithmetic intensity below the ridge point sits
under the sloped memory roof — it is memory-bound, and no amount of
additional raw compute throughput would make it faster, only more memory
bandwidth would. An operation above the ridge point sits under the flat
compute roof — it is compute-bound, and more memory bandwidth would not help
at all, only more raw compute throughput would.

## Why this is the formal version of a lesson already covered

This is exactly the framework underneath
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound), stated precisely rather than qualitatively: decode sits at
low arithmetic intensity, since each step moves the entire set of weights
through memory to perform comparatively little arithmetic on one new token,
which places it firmly under the memory roof on essentially every current
GPU. Prefill, processing many tokens of the prompt at once, achieves far
higher arithmetic intensity and typically sits under the compute roof
instead. Knowing an operation's actual arithmetic intensity, rather than
guessing which regime it falls into, is what lets an engineer predict in
advance whether a given optimisation will help at all — a change that adds
raw compute throughput does nothing for an operation that is already
memory-bound, and vice versa.
