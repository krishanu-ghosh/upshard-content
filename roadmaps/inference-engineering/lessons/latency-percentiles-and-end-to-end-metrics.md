---
title: Latency percentiles and end-to-end metrics
description: Why averages hide the real user experience — p50/p95/p99 latency, and measuring the whole request path, not just model compute.
roadmap: inference-engineering
node: latency-percentiles-and-end-to-end-metrics
group: latency-and-throughput
video:
  provider: youtube
  id: "7c9FSUVcXR0"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1190
level: advanced
prerequisites: [latency-vs-throughput]
takeaways:
  - An average latency hides exactly the worst experiences a product has, since one terrible outlier and ninety-nine fast responses can still average out to a number that looks fine.
  - Percentile latencies (p50, p95, p99) report the value below which a given fraction of requests fall, which is what actually reveals how bad the worst-affected users have it.
  - Real end-to-end latency includes everything on the request's path — routing, queueing, admission, retries — not just the time the model itself spends computing, and a system optimised only for model compute can still feel slow to users.
check_questions:
  - q: Why can a system with a good average latency still be delivering a bad experience to many users?
    a: An average blends every request together, so a small fraction of genuinely slow requests can be hidden by a larger number of fast ones. The users who happen to hit that slow fraction have a bad experience the average never reveals.
  - q: What does p99 latency actually tell you that p50 does not?
    a: P50 is the midpoint — half of requests are faster, half slower. P99 is the value only one percent of requests exceed, which is close to the worst experience a meaningful number of real users actually have, and it is often many times worse than p50.
deeper: []
related_formulas: []
---

## What an average cannot tell you

Report "average latency is 400 milliseconds" and it sounds like most users
have a snappy experience. That average could just as easily be produced by
ninety-nine requests at 200 milliseconds and one at 20 seconds — a genuinely
terrible outcome for whoever hit that one request, completely invisible in
the average. Averages are mathematically convenient and experientially
misleading, which is why production systems are measured in percentiles
instead.

## Percentiles, read correctly

A **percentile** reports the value below which a given fraction of
observations fall. **P50** (the median) is the midpoint — half of requests
are faster, half slower. **P95** is the value only five percent of requests
exceed. **P99** is the value only one percent exceed — a tail so far out
that it is easy to dismiss as rare, except at real volume: a system serving
a million requests a day still delivers ten thousand of them at or beyond
its p99, which is not a rare edge case at all once you account for scale.
Tracking p95 and p99 alongside p50 is standard practice precisely because
the median can look excellent while a meaningful slice of real users are
having a genuinely bad time.

## Two more percentiles worth naming

Beyond the overall request latency, two percentile measurements are worth
tracking in their own right. **Inter-token latency (ITL)**, sometimes called
time-per-output-token, measures the gap between consecutive tokens during
decode — its own distribution, with its own p95 and p99, separate from the
[time-to-first-token](/roadmaps/inference-engineering/time-to-first-token)
metric this group opened with. A system can have an excellent TTFT and still
deliver an uneven, stuttering stream if ITL's tail is long,
which a user perceives as the response pausing partway through rather than
streaming smoothly. **Queue time**, the portion of total latency spent simply
waiting before any processing begins, is worth isolating from processing
time specifically because the two point to entirely different fixes — a long
queue time means capacity is the problem, covered in
[autoscaling](/roadmaps/inference-engineering/autoscaling-and-cold-starts),
while long processing time points back to the model and hardware layers
covered earlier in this roadmap. Tracking both separately, rather than one
combined number, is what lets an on-call engineer tell at a glance whether a
latency spike needs more capacity or a faster model.

## End-to-end means the whole path, not just the model

It is tempting to measure "inference latency" as purely the time the model
spends computing an answer, since that is the part this roadmap spends the
most time on. In production, a request's actual latency includes everything
that happens before and after that computation: time spent being routed to
an available instance, time spent waiting in a queue behind other requests,
time spent on a retry if the first attempt failed, and the network time to
and from the client. A system can have excellent raw model-compute latency
and still feel slow to users if routing or queueing elsewhere in the stack
is adding meaningful delay — which is exactly why
[tracing the whole path](/roadmaps/inference-engineering/observability-and-cost) matters later in this roadmap, rather than instrumenting the model
call in isolation and assuming that single number tells the whole story.
