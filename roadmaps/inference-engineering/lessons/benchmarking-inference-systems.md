---
title: Benchmarking inference systems
description: Measuring a serving system's real performance under realistic load — the gap between a synthetic single-request number and actual production traffic.
roadmap: inference-engineering
node: benchmarking-inference-systems
group: serving-software
video:
  provider: youtube
  id: "OnqzoBf7dUE"
  start: 0
  end: null
creator:
  name: CNCF [Cloud Native Computing Foundation]
  channel_url: https://www.youtube.com/@cncf
duration_seconds: 1899
level: advanced
prerequisites: [inference-engines, latency-percentiles-and-end-to-end-metrics]
takeaways:
  - A single-request latency benchmark tells you almost nothing about how a system behaves under real, concurrent production traffic, where requests queue and compete for the same GPU resources.
  - A realistic benchmark varies concurrency, prompt length and output length together, since real traffic is a mix of all three rather than one fixed shape repeated identically.
  - Dedicated load-testing tools for LLM inference exist specifically because general-purpose web load testers do not model the thing that actually matters here — token-by-token streaming behaviour and the cost of a long generation, not just a single request/response round trip.
check_questions:
  - q: Why is a single-request latency benchmark a poor predictor of production performance?
    a: In production, many requests run concurrently and compete for the same GPU memory and compute, which changes latency and throughput characteristics substantially compared to a system serving one request in isolation with the whole GPU to itself.
  - q: Why do ordinary web load-testing tools fall short for benchmarking LLM serving?
    a: They are generally built around request/response round trips with a fixed, short response time in mind. LLM inference streams tokens over an extended period and its cost and latency depend heavily on output length, which a tool designed for typical web APIs does not model well.
deeper: []
related_formulas: []
---

## Why a single request tells you almost nothing

It is tempting to benchmark a serving system by sending one request and
timing how long it takes — simple, repeatable, easy to compare across
configurations. It is also close to useless for predicting production
behaviour, because a single request has the entire GPU to itself, with none
of the queueing, batching and resource contention that defines real traffic.
A system that looks excellent under this test can behave very differently
once dozens or hundreds of concurrent requests are actually competing for the
same hardware.

## What a realistic benchmark actually varies

A meaningful benchmark does not fix one request shape and repeat it; it
varies **concurrency** (how many requests are in flight simultaneously),
**prompt length** (since this directly drives prefill cost, covered in
[prefill and decode](/roadmaps/inference-engineering/prefill-and-decode)),
and **output length** (since this drives how long decode runs for a given
request) together, because real production traffic is a genuine mix of all
three rather than one representative shape. A system benchmarked only at low
concurrency with short prompts can look dramatically better than it performs
once real traffic arrives at the concurrency and prompt-length mix the
product actually sees.

## Why generic load-testing tools fall short here

Standard web load-testing tools are generally built around a request-response
model with a short, roughly fixed response time — send a request, measure
how long the response takes, repeat. LLM inference does not fit that shape:
a response streams token by token over an extended period, and its latency
and resource cost depend heavily on how many tokens it actually generates,
which varies request to request in a way a typical web endpoint's response
does not. This mismatch is why dedicated LLM-inference benchmarking tools
exist at all, measuring metrics specific to this workload — time to first
token, inter-token latency, tokens-per-second throughput at varying
concurrency — rather than the single round-trip-time number a generic tool
would report.

## Common mistakes that quietly invalidate a benchmark

A handful of pitfalls recur often enough to name directly. **Running against
a cold server**, before the first few requests have warmed up any internal
caches or just-in-time compilation, produces misleadingly slow early results
that do not reflect steady-state performance. **Testing with unrealistic
prompt lengths**, far shorter or more uniform than real traffic ever is,
understates the cost of prefill in a way that makes a system look faster
than it will actually be in production. **Ignoring warm-up time for prefix
caches**, covered in
[prefix caching](/roadmaps/inference-engineering/prefix-caching), can make a
caching-heavy system look worse than it really is if the benchmark never
gives the cache a chance to actually fill with anything reusable. Each of
these produces numbers that are internally consistent and genuinely
misleading, which is exactly why a benchmark's methodology deserves as much
scrutiny as its headline result.

## What a good benchmark actually buys you

Run properly, a benchmark answers the question
[latency versus throughput](/roadmaps/inference-engineering/latency-vs-throughput) poses directly: at a given latency target, how much throughput
can this specific configuration sustain? That answer is what lets a team
compare two inference engines, two hardware choices, or two serving
configurations on equal footing — not by whichever one posts the best number
on a synthetic single-request test, but by whichever one actually serves the
real traffic shape better, which is the only comparison that predicts what
happens once the system is live.
