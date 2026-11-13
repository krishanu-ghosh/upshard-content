---
title: Observability and cost
description: Monitoring a production inference system — the metrics that matter, attributing cost per request, and catching degradation before users do.
roadmap: inference-engineering
node: observability-and-cost
group: production
video:
  provider: youtube
  id: "wFDD6Hd_Wng"
  start: 0
  end: null
creator:
  name: PyTorch Foundation
  channel_url: https://www.youtube.com/@PyTorch
duration_seconds: 1914
level: advanced
prerequisites: [routing-and-load-balancing]
takeaways:
  - Request count, queue depth and GPU utilization alone are poor signals of real health, since a system can look busy and healthy by these measures while actually degrading in ways that matter to users.
  - Unifying latency metrics (TTFT, inter-token latency), GPU-level metrics, and KV-cache metrics into one observability view is what actually lets an engineer trace a real incident back to its cause across the whole stack this roadmap has covered.
  - Attributing cost per request, per feature, or per customer turns an opaque total GPU bill into something actionable — revealing exactly which part of a product is expensive and whether that expense is justified.
check_questions:
  - q: Why are request count and GPU utilization poor signals of whether an inference system is actually healthy?
    a: A system can show high utilization and steady request counts while quietly degrading in ways that matter — rising latency, a growing queue, or memory pressure building toward an eventual failure — none of which a simple utilization or request-count graph reveals on its own.
  - q: Why does attributing cost per request or per feature matter more than watching a single total GPU bill?
    a: A single aggregate bill tells you spending went up, but not why, or whether the increase is justified. Breaking cost down by request, feature, or customer reveals exactly which part of the product is driving the expense, which is what actually lets a team decide whether to optimize it, price for it, or accept it.
deeper:
  - label: "Momento — Building Reliable LLM Serving Infrastructure at Scale"
    url: https://www.youtube.com/watch?v=cCfiJN2lfaw
  - label: "Momento — Reliability Engineering for Inference Serving"
    url: https://www.youtube.com/watch?v=vezhxTtrZAg
related_formulas: []
---

## The signals that look fine right up until they are not

A dashboard showing steady request counts and healthy-looking GPU
utilization can coexist with a system that is quietly getting worse: latency
creeping upward, a queue slowly growing, memory pressure building toward an
eventual out-of-memory failure. None of the obvious, easy-to-watch metrics
reveal this kind of gradual degradation on their own, which is why
observability for an inference system needs to go beyond the surface-level
numbers that are easiest to graph.

## Unifying the metrics this roadmap has covered

The useful signals here are the ones covered throughout this roadmap, brought
together into one view rather than scattered across separate dashboards:
[TTFT](/roadmaps/inference-engineering/time-to-first-token) and inter-token
latency at the
[percentiles that actually matter](/roadmaps/inference-engineering/latency-percentiles-and-end-to-end-metrics), GPU utilization and memory pressure from
[the hardware layer](/roadmaps/inference-engineering/gpu-architecture-for-inference), and KV cache occupancy from
[cache mechanics](/roadmaps/inference-engineering/kv-cache-mechanics).
Tracing a real incident — why did latency spike at 3am — means being able to
follow the thread across all of these at once: was it a traffic spike that
exceeded capacity, a cold start that took too long, a routing decision that
sent too much load to one instance, or memory pressure from an
unexpectedly long context? Without a unified view across the whole stack,
answering that question means guessing rather than tracing.

## Cost attribution: turning a bill into a decision

A single aggregate GPU bill tells a team that spending rose, but nothing
about why or whether that rise is justified. Attributing cost per request,
per product feature, or per customer turns that opaque total into something
genuinely actionable: if one feature accounts for a disproportionate share
of spend, that is either a signal to optimise it specifically — perhaps it
is using a larger model than it needs, per
[choosing hardware](/roadmaps/inference-engineering/choosing-hardware), or
missing an obvious
[caching](/roadmaps/inference-engineering/prefix-caching) opportunity — or a
signal that the feature is simply worth its cost and the number is fine as
is. Either conclusion requires the breakdown; the aggregate total alone
supports neither.

## Alerting on the derivative, not just the level

A single threshold alert — fire when latency exceeds some fixed number — is
a reasonable starting point and frequently not enough on its own. Cost in
particular is better watched for its rate of change than its absolute level:
a GPU bill that is high but stable is a known, budgeted cost, while one that
is climbing day over day with no corresponding change in traffic is a signal
something has gone wrong, whether a runaway process, a routing misconfiguration sending traffic somewhere expensive it should not be, or a quality
regression causing retries to spike. Watching the derivative — is this
getting worse, and how fast — catches problems while they are still small,
well before an absolute threshold would have been crossed at all.

## Catching problems before users report them

The entire point of good observability here is shifting from reactive to
proactive: discovering a quality or performance regression through a
dashboard alert, rather than through a flood of user complaints after the
fact. The deeper links cover this from two angles worth reading together —
building serving infrastructure reliable enough that these incidents are
rarer in the first place, and the reliability-engineering practices for
inference serving once something does go wrong regardless of how well the
system was built.
