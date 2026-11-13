---
title: Cloud ML platforms
description: What managed platforms like SageMaker and Vertex AI actually buy you, and the real tradeoff against building the equivalent on raw infrastructure.
roadmap: mlops
node: cloud-ml-platforms
group: orchestration-infra
video:
  provider: youtube
  id: "gT4qqHMiEpA"
  start: 0
  end: null
creator:
  name: Google Cloud Tech
  channel_url: https://www.youtube.com/@googlecloudtech
duration_seconds: 436
level: intermediate
prerequisites: [infrastructure-as-code]
takeaways:
  - A managed ML platform bundles compute provisioning, training orchestration, model registries, and serving infrastructure behind one API, trading cost and flexibility for speed and reduced operational burden.
  - The build-vs-buy decision isn't about team size or budget alone — it's about whether your workload's needs sit inside the platform's supported patterns or require control the platform deliberately abstracts away.
  - Lock-in with a managed platform is real but gradual and mostly about operational tooling and workflow, not about the model artifacts themselves, which usually remain portable.
check_questions:
  - q: What does a managed ML platform like SageMaker or Vertex AI provide that raw cloud infrastructure (VMs, storage, networking) does not?
    a: Pre-built, integrated services for the ML lifecycle specifically — managed training jobs, a model registry, one-click or API-driven deployment to a scalable serving endpoint, and built-in monitoring — so a team doesn't have to assemble and operate each of those from general-purpose infrastructure.
  - q: Under what circumstances does building your own ML infrastructure tend to beat using a managed platform?
    a: When the workload has unusual scale, custom hardware, or orchestration requirements the platform doesn't support well, when the team already has the platform engineering expertise, or when the ongoing premium of managed services outweighs the engineering time it saves at that team's scale.
  - q: What is actually locked in when a team builds deeply on a managed ML platform, and what usually isn't?
    a: The operational workflow, pipeline definitions, and platform-specific tooling become hard to migrate away from, but the trained model artifacts themselves are typically standard formats that can run elsewhere, so the lock-in is mainly about process, not the model.
deeper:
  - label: "Introduction to SageMaker AI — Amazon Web Services"
    url: "https://www.youtube.com/watch?v=ZBb3ZtajCdI"
related_formulas: []
---

## What a managed platform actually bundles

A cloud ML platform like SageMaker or Vertex AI isn't one product — it's a
bundle of services that would otherwise be separate things you'd have to
provision and wire together yourself: managed training jobs that handle
GPU allocation and distributed setup, a model registry for versioning
trained artifacts, one-click or API-driven deployment to an autoscaling
serving endpoint, built-in experiment tracking, and monitoring for drift
and performance once a model is live. The appeal is that these pieces are
pre-integrated. Submit a training job and the platform handles spinning up
the right instance type, mounting the data, running the job, and tearing
the compute down when it finishes — work that otherwise requires writing
and maintaining IaC definitions, orchestration DAGs, and serving
infrastructure by hand, as the two previous lessons in this group cover.
The platform is, in effect, someone else's already-built and
already-operated version of all of that.

## The tradeoff is speed and safety versus cost and control

The honest framing of build-vs-buy here isn't "managed platforms are for
small teams and custom infrastructure is for big ones" — that's too
simple, and plenty of large, sophisticated ML orgs run heavily on managed
platforms. The real tradeoff is along two axes. Speed and operational
safety: a managed platform gets a team from trained model to a working,
autoscaled, monitored endpoint in days, run by a small team, with the
platform vendor responsible for the serving infrastructure staying up.
Cost and control: that convenience carries a real premium over raw compute
pricing, and it caps what you can customize — if your workload needs an
unusual distributed training topology, custom hardware the platform
doesn't support, or serving latency characteristics the managed endpoint
can't hit, you're fighting the platform's abstractions instead of being
helped by them.

## A concrete way to decide

A useful test: can you describe your training and serving requirements
using the platform's supported job types, instance types, and deployment
patterns without working around them? If yes, the platform likely saves
real engineering time, because you're paying for infrastructure work you
would otherwise have to build and, more importantly, operate — keep
patched, keep scaled, keep monitored — indefinitely. If your requirements
routinely fall outside what the platform supports — custom accelerators,
nonstandard parallelism strategies, extremely latency-sensitive serving
that needs infrastructure control the platform abstracts away — building
on raw infrastructure (using the IaC and orchestration tools from the
earlier lessons in this group) gives you that control, at the cost of now
owning the operational burden the platform would otherwise have carried.
Team size matters here mostly as a proxy for whether you *have* the
platform engineering capacity to own that burden well, not as a rule on
its own.

## Lock-in is real, but not where people usually point

The common worry is "if we build on SageMaker, we're stuck with AWS
forever." The sharper version of that worry is more useful: what's
actually locked in is the operational workflow — pipeline definitions
expressed in the platform's SDK, deployment configurations tied to its
specific APIs, monitoring dashboards built on its specific tooling. A
trained model itself is usually a standard artifact (an ONNX file, a
PyTorch checkpoint, a container image) that can, in principle, run
somewhere else with some rework. Migrating off a platform means rebuilding
the surrounding workflow, not retraining the model from scratch. That's
still a real cost — often underestimated — but it's a different, smaller
cost than the all-or-nothing framing suggests, and it's worth evaluating
against honestly rather than treating lock-in as an automatic
disqualifier.

## Why this matters going forward

Cloud ML platforms are the point where the infrastructure-as-code and
orchestration disciplines from the earlier lessons meet a genuine business
decision: build the equivalent yourself, with full control and full
operational responsibility, or rent an integrated version of it and accept
its boundaries. Neither answer is correct in general — it depends on
whether your workload fits inside what the platform supports and whether
your team would rather spend its time on infrastructure or on the model
itself. The next lesson, on distributed training infrastructure, is exactly
the kind of requirement that can push that decision one way or the other:
some distributed training setups are well inside what managed platforms
handle cleanly, and some aren't.
