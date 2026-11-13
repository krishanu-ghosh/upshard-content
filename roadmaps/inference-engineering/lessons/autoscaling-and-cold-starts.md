---
title: Autoscaling and cold starts
description: Scaling GPU capacity with demand, and the real cost of a cold start — how long it takes a new instance to load a large model before it can serve anything.
roadmap: inference-engineering
node: autoscaling-and-cold-starts
group: production
video:
  provider: youtube
  id: "EXQ3AksFXcM"
  start: 0
  end: null
creator:
  name: Koyeb
  channel_url: https://www.youtube.com/@gokoyeb
duration_seconds: 1221
level: advanced
prerequisites: [benchmarking-inference-systems, disaggregated-serving]
takeaways:
  - A cold start is the time a new serving instance takes to become ready — loading a large model's weights from storage into GPU memory before it can answer a single request.
  - For a large model, a cold start can take minutes, which makes naive autoscaling (spin up a new instance the moment demand rises) far too slow to actually respond to a sudden traffic spike.
  - Streaming weight loading, keeping a pool of warm standby instances, and other techniques covered in this lesson all exist specifically to shrink the gap between demand arriving and capacity being ready to serve it.
check_questions:
  - q: Why can a large model's cold start take minutes rather than seconds?
    a: The model's entire set of weights — potentially hundreds of gigabytes — has to be read from storage and loaded into GPU memory before the instance can process a single request, and that read-and-load process is itself a substantial amount of data movement that takes real time regardless of how fast the GPU itself is once running.
  - q: Why does a multi-minute cold start make naive autoscaling ineffective against a sudden traffic spike?
    a: By the time a newly spun-up instance finishes loading and becomes ready to serve, several minutes of the spike may have already passed, during which existing capacity was overwhelmed and users experienced degraded service. The new capacity arrives too late to actually absorb the spike it was meant to handle.
deeper:
  - label: "CNCF — Optimizing Model Serving on Kubernetes With Model Streaming"
    url: https://www.youtube.com/watch?v=qH5djJlbodY
  - label: "Modal — Truly Serverless GPUs: A Deep Dive Inside Modal's Fast Cold Starts"
    url: https://www.youtube.com/watch?v=G3M46cpdF4g
related_formulas: [model-memory-footprint]
---

## Capacity that has to actually be ready, not just requested

Scaling a web service typically means starting a new, lightweight process
that is ready to handle requests within moments. Scaling GPU inference
capacity is a fundamentally slower operation, because a new instance is not
actually useful until it has loaded the model's entire set of weights — per
[model memory footprint](/formulas/model-memory-footprint), potentially
hundreds of gigabytes — from storage into GPU memory. This loading time, the
**cold start**, is the central problem this lesson covers, and it changes
what "autoscaling" has to mean for inference compared to ordinary web
infrastructure.

## Why a slow cold start defeats naive autoscaling

The simplest autoscaling strategy — watch demand, and spin up a new instance
the moment it rises — assumes new capacity becomes available quickly enough
to matter. For a large model with a multi-minute cold start, this assumption
breaks down: by the time a newly requested instance finishes loading and
becomes ready to serve, the traffic spike that triggered it may already have
overwhelmed existing capacity for several minutes, with users experiencing
degraded service the whole time the new instance was still loading. The new
capacity arrives, correctly provisioned, but too late to have actually
helped with the problem it was meant to solve.

## Techniques that shrink the gap

Several approaches exist to close the gap between demand arriving and
capacity being ready. **Streaming weight loading** begins making an instance
partially usable before every single weight has finished loading, rather
than waiting for the complete load to finish before serving anything at all.
**Keeping a pool of warm standby instances** — already loaded and ready,
sitting idle until needed — trades a real ongoing cost (paying for GPU
capacity that may not be used) for the ability to absorb a spike instantly,
without waiting on any cold start at all. **Predictive scaling**, where
traffic patterns are known or forecastable in advance, can start loading new
capacity before demand actually arrives, converting what would have been a
reactive cold start into one that finishes before it is needed. None of these
techniques eliminates the cold start outright; each trades something else —
engineering complexity, idle cost, or forecasting accuracy — for a shorter
effective gap, and a production system typically combines more than one of
them rather than relying on a single fix.

## Why this connects directly to the economics covered elsewhere

A cold start is not only a latency problem — it is directly a cost problem
too, since keeping enough warm standby capacity to avoid cold starts entirely
means paying for GPU time that sits idle between traffic spikes. Deciding how
much standby capacity is worth maintaining, against how often and how
severely cold starts would otherwise hurt users, is exactly the kind of
tradeoff covered in
[observability and cost](/roadmaps/inference-engineering/observability-and-cost) later in this roadmap — there is no universally correct answer, only
a deliberate choice informed by your product's actual traffic patterns and
its tolerance for occasional slow responses during a genuine spike.
