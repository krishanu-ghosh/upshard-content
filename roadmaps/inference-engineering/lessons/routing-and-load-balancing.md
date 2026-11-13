---
title: Routing and load balancing
description: Directing a request to the right instance — accounting for which model or adapter is loaded, current queue depth, and which serving phase an instance handles.
roadmap: inference-engineering
node: routing-and-load-balancing
group: production
video:
  provider: youtube
  id: "sOB3HSiG8vo"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1092
level: advanced
prerequisites: [autoscaling-and-cold-starts, prefix-caching]
takeaways:
  - Ordinary round-robin load balancing, which ignores what each instance is actually doing, performs badly for LLM inference because instances are not interchangeable the way stateless web servers are.
  - A good router accounts for engine-level signals — current queue depth, what is already cached on each instance, whether a disaggregated setup needs prefill or decode specifically — rather than treating every instance as identical.
  - Routing a request to an instance that already has the relevant prefix cached, rather than an idle instance with nothing cached, can be faster even if the cached instance currently has more load.
check_questions:
  - q: Why does plain round-robin load balancing perform poorly for LLM inference specifically?
    a: Round-robin assumes every instance is interchangeable and in the same state, which is true for simple stateless web servers but not for inference instances — they differ in current queue depth, what is cached in their KV cache, and in a disaggregated setup, which phase of serving they even handle.
  - q: Why might routing to a busier instance with a relevant cached prefix beat routing to a completely idle instance?
    a: The busier instance can reuse its already-computed KV cache for the shared prefix via prefix caching, skipping that portion of prefill entirely, while the idle instance would have to compute the entire prefix from scratch. The time saved by reusing the cache can outweigh the cost of some additional queueing.
deeper:
  - label: "CNCF — Keynote: LLM-Aware Load Balancing in Kubernetes"
    url: https://www.youtube.com/watch?v=BBqDpqATcI0
  - label: "The Linux Foundation — Beyond Round-Robin: GPU-Aware Load Balancing"
    url: https://www.youtube.com/watch?v=sVBWCp8W1WY
related_formulas: []
---

## Why instances are not interchangeable

Ordinary web load balancing often assumes every backend instance is
interchangeable — stateless, equally capable of handling any request, so
routing decisions can be as simple as round-robin or least-connections.
Inference instances break this assumption in several real ways: one instance
might already have a request's relevant prompt prefix cached, per
[prefix caching](/roadmaps/inference-engineering/prefix-caching), while
another has nothing cached at all; one might be nearly saturated while
another sits mostly idle; and in a
[disaggregated serving](/roadmaps/inference-engineering/disaggregated-serving) deployment, instances are not even running the same job — some handle
prefill exclusively, others handle decode exclusively. Routing that ignores
all of this performs noticeably worse than routing that accounts for it.

## What a smarter router actually considers

A router built for inference specifically can use several engine-level
signals a generic load balancer never sees: **current queue depth** at each
instance, to avoid piling more work onto an already-saturated one;
**cache state**, to prefer an instance that already has a relevant prefix
cached even if it means slightly more queueing, since the cache hit can save
more time than the extra wait costs; and, in a disaggregated deployment,
**which phase** a given instance actually serves, so a prefill request is
never mistakenly sent to a decode-only instance. None of these signals exist
in a typical stateless web service, which is exactly why inference routing
needs its own dedicated approach rather than reusing generic load-balancing
tooling unchanged.

## The specific case of cache-aware routing

Worth dwelling on directly, because it is the least intuitive of these
signals: routing a request to a busier instance over an idle one can be the
*faster* choice, specifically when the busier instance already has the
request's prompt prefix cached. The idle instance would have to compute that
entire prefix from scratch, while the busy one only has to process whatever
is genuinely new, even after accounting for some additional queueing delay.
This is a direct, practical consequence of everything
[prefix caching](/roadmaps/inference-engineering/prefix-caching) makes
possible, and it is exactly the kind of routing decision a generic,
cache-unaware load balancer has no way to make correctly.

## What happens when routing gets it wrong

A poorly tuned router produces two visible failure patterns worth
recognising. **Hot-spotting** happens when too many requests land on the same
instance — because it has a popular cached prefix, say — while other
instances sit comparatively idle, which looks like healthy average
utilisation on a dashboard while individual users experience very uneven
latency depending on which instance they happened to land on. **Thrashing**
happens when a router switches an instance's workload too aggressively,
constantly swapping which model or adapter is loaded in response to
short-term signals, paying the real cost of a reload or cache eviction more
often than the routing decision actually saves. Both are symptoms of a router
optimising too narrowly for one signal — cache affinity alone, or queue
depth alone — rather than balancing several signals together, which is the
real engineering difficulty in building a good one.

## Routing for adapters, and routing at scale

Where multiple fine-tuned variants of a base model — covered in Upshard's
Machine Learning and AI Engineer roadmaps as LoRA adapters — are served off
shared infrastructure, routing also has to account for which adapter is
currently loaded on which instance, since switching an instance's active
adapter has its own cost. The deeper links cover the Kubernetes-specific
tooling built around exactly these signals — queue depth, cache affinity,
adapter availability — for teams running inference at a scale where manual
routing decisions are no longer practical to make by hand.
