---
title: Scaling inference
description: Autoscaling, batching, caching, and CPU vs. GPU choices are the levers that keep a model serving fast and cheap once real traffic hits it.
roadmap: mlops
node: scaling-inference
group: serving-deployment
video:
  provider: youtube
  id: "Nw77sEAn_Js"
  start: 0
  end: null
creator:
  name: Databricks
  channel_url: https://www.youtube.com/@Databricks
duration_seconds: 996
level: advanced
prerequisites: [deployment-strategies, cloud-ml-platforms]
takeaways:
  - Autoscaling, request batching, and caching are independent levers that each solve a different bottleneck, and most production serving systems need more than one of them at once.
  - Batching trades a small amount of added latency per request for much higher hardware throughput, which is why it matters so much more for GPU-backed models than for small CPU ones.
  - Choosing CPU vs. GPU isn't about which is "better" — it's about whether the model's compute profile and traffic pattern can actually make use of a GPU's parallelism.
check_questions:
  - q: Why does request batching improve throughput on a GPU specifically?
    a: Because a GPU is built for massive parallel computation, so running several requests through the model in a single batched pass uses far more of its capacity per unit time than running requests one at a time.
  - q: What's the core tradeoff autoscaling has to balance for inference workloads?
    a: Scaling up fast enough to absorb a traffic spike without dropping or delaying requests, against not over-provisioning idle capacity that costs money while sitting unused.
  - q: When does caching inference results stop being useful?
    a: When inputs are highly unique per request, since a cache only helps when the same or similar inputs recur often enough that a stored prediction can be reused instead of recomputed.
deeper: []
related_formulas: []
---

## Serving one request is easy; serving all of them is the job

Getting a model to return a correct prediction for one request is the
easy part — that's model development. Scaling inference is the much
harder, separate problem of keeping that same model fast, available, and
affordable once real production traffic shows up: thousands of requests
a second, arriving unevenly, from users who won't wait around for a slow
response. Four levers do most of the work here — autoscaling, request
batching, caching, and the choice of hardware — and they solve different
bottlenecks, which is why production systems typically need several of
them working together rather than any single one.

## Autoscaling: matching capacity to a moving target

Traffic to a serving endpoint is rarely flat. It spikes around product
launches, follows daily and weekly usage cycles, and sometimes jumps
unpredictably. Autoscaling adds or removes serving instances based on
live load — request rate, queue depth, or GPU/CPU utilization — so you're
not paying for peak-traffic capacity around the clock, and not getting
overwhelmed when traffic exceeds whatever fixed capacity you provisioned.

The hard part isn't the scaling policy itself, it's the lag between
demand rising and new capacity becoming available. A new instance serving
a large model typically needs the model weights loaded into memory before
it can accept traffic — that can take anywhere from seconds to minutes,
during which requests are still arriving. Scale too conservatively and
users hit timeouts during the ramp-up; scale too aggressively and you're
paying for idle GPUs most of the day. Good autoscaling for inference
usually means scaling on a leading indicator (queue depth growing) rather
than a lagging one (current utilization already maxed out), precisely to
buy back that lead time.

## Batching: trading a little latency for a lot of throughput

Modern inference hardware, especially GPUs, is built around massive
parallelism — it's efficient at doing one large matrix operation, not at
doing many tiny ones back to back. Running requests through a model one
at a time, as they arrive, leaves most of that parallel capacity unused.
Request batching collects several incoming requests over a short window
(a few milliseconds) and runs them through the model together as one
batch, which uses the hardware far more efficiently and dramatically
increases total throughput.

The cost is latency: a request that arrives first in a batching window has
to wait for the window to fill (or time out) before it gets processed,
adding a small delay it wouldn't have had if it were served alone. This is
a genuine tradeoff, not a free win — the batch window size is a tunable
parameter, and the right value depends on your latency budget. A
fraud-check endpoint with a tight latency SLA might use a tiny window or
skip batching entirely; a bulk document-scoring endpoint can use a wider
window and gain far more throughput per dollar of hardware.

## Caching: skip the model when you've already seen the answer

Not every request needs to hit the model. If the same or similar inputs
recur — a popular product's embedding, a common search query, a
frequently-translated phrase — caching the prediction and serving it
again on a repeat input avoids running inference at all for that request.
This is the cheapest possible latency win when it applies, because a
cache lookup is orders of magnitude faster than a model forward pass.

Caching's value depends entirely on how often inputs actually repeat.
A recommendation system serving the same small catalog of items to many
users can cache aggressively; a fraud model scoring unique transactions
has almost nothing to cache, because no two inputs are the same. Knowing
which kind of workload you have is what determines whether caching is
worth building at all.

## CPU vs. GPU: match the hardware to the compute profile

GPUs excel at the kind of large parallel matrix math that deep neural
networks need, but they're expensive, and spinning them up has overhead.
A small model, or a workload with low request volume, often runs
perfectly well — and more cheaply — on CPU, where the parallelism a GPU
offers would mostly go unused. GPUs earn their cost when the model is
large enough, and the traffic volume (especially combined with batching)
is high enough, to actually saturate that parallel capacity. Defaulting
to GPU because it sounds faster, without checking whether the workload
can use it, is a common way to overspend on inference infrastructure.

## Carrying this forward

None of these four levers replaces the others — a production system
handling meaningful load typically autoscales to track demand, batches
requests to use its hardware efficiently, caches what it can to avoid
unnecessary work, and picks hardware matched to its actual compute
profile. The next lesson shifts to a different end of the spectrum
entirely: what changes when the model has to run on the device itself,
with no server to scale or batch against at all.
