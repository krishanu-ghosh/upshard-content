---
title: Testing and safe deployment
description: Rolling out a new model version or serving configuration without downtime, and catching a regression before it reaches all production traffic.
roadmap: inference-engineering
node: testing-and-safe-deployment
group: production
video:
  provider: youtube
  id: "J1p0DvC-w1Q"
  start: 0
  end: null
creator:
  name: AWS Events
  channel_url: https://www.youtube.com/@AWSEventsChannel
duration_seconds: 1782
level: advanced
prerequisites: [observability-and-cost]
takeaways:
  - Shadow testing runs a new model or configuration against copied real traffic without affecting what users actually see, which is the lowest-risk way to observe how a change behaves on genuine production inputs.
  - A canary deployment sends a small, deliberately limited slice of real traffic to a new version, watching closely before gradually widening that slice rather than switching everyone over at once.
  - LLM-specific regressions — a quantization scheme that quietly degrades quality, a model update that shifts latency or cost — need the same careful, staged rollout discipline as any other production change, arguably more so given how easy a quality shift is to miss without deliberate testing for it.
check_questions:
  - q: Why is shadow testing considered the lowest-risk way to evaluate a change before it reaches users?
    a: The new version processes real, copied production traffic, but its output is never actually shown to any user, so a bug or regression in the new version has no chance of affecting anyone's real experience while it is being observed.
  - q: What is the risk of switching all traffic to a new model version at once instead of using a canary?
    a: Any regression the new version has — a quality drop, a latency increase, an unexpected cost spike — immediately affects every single user rather than a small, contained slice, and there is no early warning before the full blast radius is already realized.
deeper:
  - label: "Nishanth Reddy — Deploy ML Models Safely on Amazon SageMaker AI"
    url: https://www.youtube.com/watch?v=0_GHBL4KqWM
related_formulas: []
---

## Changing a running system without breaking it

Every other lesson in this roadmap covers getting an inference system
running well. This one covers changing it safely once it is already live and
serving real traffic — a new model version, a different quantization
scheme, an updated serving configuration — without a bad change reaching
every user at once, or the system going down while the change rolls out.

## Shadow testing: watch without affecting anyone

**Shadow testing** sends a copy of real, live production traffic to a new
version running alongside the current one, without ever showing that new
version's output to any actual user. This is the lowest-risk way to observe
how a change behaves against genuine traffic patterns — real prompt lengths,
real context, real edge cases that a synthetic test might never think to
include, per the same reasoning covered in
[benchmarking inference systems](/roadmaps/inference-engineering/benchmarking-inference-systems) — because whatever the new version does, correct or
not, never reaches a real user while it is being observed this way.

## Canary deployment: a small, deliberate slice

Once shadow testing has given some confidence, a **canary deployment** sends
a small, deliberately limited fraction of real traffic to the new version —
commonly starting as low as one percent, or even lower for a particularly
high-stakes change — and widens that fraction gradually as confidence grows,
rather than switching every user over in one step. If something goes wrong,
only the canary's limited slice of users is affected, and the rollout can be
halted and reverted before the blast radius grows any further. This is
exactly the same staged-exposure principle used throughout software
deployment generally, applied here to a model or serving-configuration
change specifically.

## What is specific to rolling out an LLM serving change

A regression in a traditional ML model endpoint usually shows up as a clear
error or an obviously wrong prediction. An LLM serving regression can be far
subtler: a new quantization scheme might degrade output quality in ways that
are not obviously wrong, only slightly worse — missed by a shallow smoke test
but real and measurable at scale. This is exactly why the
[eval sets and quality metrics](/roadmaps/inference-engineering/observability-and-cost) covered in the previous lesson have to be part of what a canary
rollout actually checks, not just latency and error rate: a canary that only
watches for crashes and slow responses can sail straight past a genuine
quality regression that a dedicated quality check would have caught
immediately.

## Where this leaves the whole roadmap

Testing and safe deployment is the last stage of the loop this roadmap has
built toward across every group: choose hardware and a serving stack, tune it
for speed and cost, monitor it in production, and change it carefully when
something needs to improve — then watch the result and repeat. Nothing in
production inference is ever finished in the sense a training run finishes;
it is operated, observed, and improved continuously, which is the discipline
this entire roadmap has been describing from its very first lesson onward.
