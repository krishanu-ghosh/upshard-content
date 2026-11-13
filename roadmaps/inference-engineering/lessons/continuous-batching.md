---
title: Continuous batching
description: Dynamically adding and removing requests from a running batch as they arrive and finish, keeping GPUs busy under real, bursty traffic.
roadmap: inference-engineering
node: continuous-batching
group: speeding-up-inference
video:
  provider: youtube
  id: "oObhZfVdcpI"
  start: 0
  end: null
creator:
  name: AI21 Labs
  channel_url: https://www.youtube.com/@ai21labs
duration_seconds: 1409
level: advanced
prerequisites: [inference-engines, latency-vs-throughput]
takeaways:
  - Static batching waits for a fixed group of requests to all finish together before starting the next batch, wasting GPU capacity whenever one request in the batch finishes earlier than the others.
  - Continuous batching instead adds a newly arrived request into a running batch and removes a finished one the moment it completes, at the level of individual decode steps rather than whole requests.
  - This keeps the GPU doing useful work far more consistently under real, bursty traffic, where requests of wildly different lengths arrive and finish at unpredictable times.
check_questions:
  - q: What specifically does static batching waste, and why?
    a: It wastes GPU capacity on any request in the batch that finishes generating before the others, since the GPU has to wait for the longest-running request in that batch to complete before starting the next batch — the slots freed up by shorter requests sit idle in the meantime.
  - q: At what granularity does continuous batching make its add-and-remove decisions?
    a: At the level of individual decode steps, not whole requests — a finished request can be removed and a new one added into the batch between one generation step and the next, rather than waiting for every request in the current batch to finish together.
deeper:
  - label: "Databricks — Accelerating LLM Inference with vLLM"
    url: https://www.youtube.com/watch?v=qBFENFjKE-M
related_formulas: [littles-law]
---

## The waste static batching cannot avoid

The simplest way to batch requests together is **static batching**: collect
a fixed group of requests, run them all through decode together step by
step, and only start the next batch once every request in the current one
has finished generating. The problem is that real requests rarely finish at
the same time — one might generate a short answer in a dozen tokens while
another runs for hundreds. Under static batching, the GPU slots freed up by
the shorter requests sit idle for the remainder of the batch, waiting on
whichever request happens to run longest, which wastes exactly the capacity
[batching](/roadmaps/inference-engineering/tokens-per-second-and-throughput)
was meant to use efficiently in the first place.

## Continuous batching: add and remove at the step level

**Continuous batching** removes this waste by operating at a finer
granularity: rather than waiting for an entire batch to finish together, it
adds a newly arrived request into the running batch and removes a finished
one the moment it completes, at the level of individual decode steps. A slot
freed by a request that just finished is immediately available for the next
waiting request, rather than sitting idle until the whole batch clears. This
keeps the GPU doing useful work far more consistently under real traffic,
where request lengths vary unpredictably and arrive at unpredictable times
rather than as a conveniently uniform group.

## A small worked trace

Picture a batch of four requests, A through D, each generating a different
number of tokens: A needs 5, B needs 40, C needs 12, D needs 60. Under static
batching, the GPU runs all four together for 60 steps, since nothing can leave
until D finishes — A, B and C each sit idle in their slot for most of that
time after they are done. Under continuous batching, A's slot frees up after
step 5 and a new request, E, steps in immediately; C's slot frees after step
12 and another waiting request takes it. By the time D finishes at step 60,
the GPU has served several more requests in the slots that would otherwise
have sat empty, with the same total compute budget.

## Why this matters more as traffic gets more realistic

The gap between static and continuous batching grows with how varied real
traffic actually is. A workload where every request happens to be nearly
identical in length sees little benefit from continuous batching, since
static batching's waste is small when nothing finishes early. Real
production traffic is essentially never this uniform — request lengths vary
by user, by task, by how much context is attached — which is exactly why
continuous batching has become close to a baseline expectation in any
serious inference engine rather than an optional optimisation, and why every
engine covered in
[inference engines](/roadmaps/inference-engineering/inference-engines)
implements some form of it.

## Connecting back to the throughput-latency tradeoff

Continuous batching does not eliminate the fundamental
[latency-versus-throughput tradeoff](/roadmaps/inference-engineering/latency-vs-throughput) covered earlier in this roadmap — a batch that is too large
still slows down any individual request's per-token speed, exactly as before.
What it changes is efficiency at a given batch size: for the same number of
concurrent requests in flight, continuous batching extracts substantially
more useful throughput from the hardware than static batching would, by
never leaving a freed slot idle. [Little's Law](/formulas/littles-law) still
governs how many requests a system can hold in flight at once; continuous
batching is what makes sure that capacity is actually used rather than
wasted.
