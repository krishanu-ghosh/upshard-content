---
title: Disaggregated serving
description: Separating the prefill and decode phases onto different hardware pools, since they have such different performance characteristics.
roadmap: inference-engineering
node: disaggregated-serving
group: scaling-and-beyond-llms
video:
  provider: youtube
  id: "3C-6STonTLU"
  start: 0
  end: null
creator:
  name: NVIDIA Developer
  channel_url: https://www.youtube.com/@NVIDIADeveloper
duration_seconds: 5358
level: advanced
prerequisites: [prefill-and-decode, model-parallelism]
takeaways:
  - Prefill is compute-bound and bursty, while decode is memory-bound and sustained, which means a single machine serving both phases together is poorly matched to either one.
  - Disaggregated serving runs prefill on one pool of hardware and decode on another, each provisioned and scaled for its own specific resource profile rather than compromising for both.
  - The two pools communicate by transferring the KV cache computed during prefill over to whichever machine handles decode for that request, which introduces its own latency cost that has to be kept small relative to the gains.
check_questions:
  - q: Why is a single machine serving both prefill and decode a compromise for both phases?
    a: Prefill wants hardware optimized for raw compute throughput and handles bursty demand, while decode wants hardware optimized for memory bandwidth and runs as a sustained, steady load. A machine configured as a compromise between these two different profiles serves neither phase as well as hardware dedicated to just one would.
  - q: What new cost does disaggregating prefill and decode introduce that a combined machine does not have?
    a: The KV cache computed during prefill has to be transferred from the prefill machine to whichever decode machine will continue generating that request, over a network connection — a transfer cost that a single machine handling both phases never has to pay at all, since the cache never has to leave the device.
deeper: []
related_formulas: []
---

## Two phases that want different hardware

[Prefill and decode](/roadmaps/inference-engineering/prefill-and-decode)
established that these two phases of generating a response have almost
opposite resource profiles: prefill is compute-bound, processing an entire
prompt in one burst of parallel work; decode is memory-bandwidth-bound,
running as a sustained, steady sequence of smaller steps for as long as
generation continues. A single machine configured to serve both phases
together is, by construction, a compromise — whatever balance of compute and
memory bandwidth it strikes serves neither phase as well as hardware
dedicated to just one of them would.

## Disaggregation: serve each phase on hardware built for it

**Disaggregated serving** takes this observation to its logical conclusion:
run prefill on one pool of GPUs, provisioned and scaled for prefill's
bursty, compute-heavy demand, and run decode on an entirely separate pool,
provisioned and scaled for decode's sustained, memory-bandwidth-heavy
demand. Each pool can be sized, and even chosen on different hardware
entirely, to match exactly what its one job actually needs, rather than
forcing a single configuration to serve both reasonably rather than either
one well.

## Sizing the two pools independently

Because prefill and decode now run on separate pools, each pool's size no
longer has to track the other's. A workload with many short conversations and
few long ones might need a large decode pool, sized for sustained
memory-bandwidth demand across many concurrent sessions, alongside a
comparatively small prefill pool, since any one prompt only occupies it
briefly. A workload dominated by long documents processed once each needs the
opposite ratio. Under combined serving, both phases are forced to share
whatever ratio of compute to memory bandwidth the single chosen machine
happens to have; disaggregation lets the ratio of prefill to decode capacity
track the actual workload instead.

## The cost disaggregation introduces

Splitting the two phases onto separate hardware is not free: the KV cache
computed during prefill, per
[KV cache mechanics](/roadmaps/inference-engineering/kv-cache-mechanics), has
to actually move from the prefill machine to whichever decode machine will
continue generating that specific request, over a network connection between
them. This transfer takes real time, and it is a cost a single combined
machine never has to pay at all, since the cache never has to leave the
device it was computed on. Disaggregation only pays off when the efficiency
gained from giving each phase hardware matched to its own needs exceeds this
transfer cost — which is generally true at real production scale, where the
efficiency gains are substantial, but is a genuine engineering tradeoff
rather than a free win.

## Where this fits in a full serving architecture

Disaggregated serving is usually combined with the
[model parallelism](/roadmaps/inference-engineering/model-parallelism)
techniques from the previous lesson rather than replacing them — a large
model might still need tensor parallelism within each individual prefill or
decode machine, with disaggregation operating at the level of deciding which
pool of machines handles which phase at all. This combination of
techniques — splitting a model across GPUs where it does not fit on one, and
splitting the two phases of serving it across differently-provisioned
pools — is what large-scale production serving for the biggest current
models actually looks like, well beyond what a single machine running a
single copy of an inference engine, covered earlier in this roadmap, is
built to handle.
