---
title: What is inference engineering?
description: Running a trained model efficiently in production — a different job from training one, and a discipline that exists because serving at scale got hard.
roadmap: inference-engineering
node: what-is-inference-engineering
group: foundations
video:
  provider: youtube
  id: "esoZ-u5eqeU"
  start: 0
  end: null
creator:
  name: CoreWeave
  channel_url: https://www.youtube.com/@coreweave
duration_seconds: 1290
level: advanced
prerequisites: []
takeaways:
  - Inference engineering is the discipline of running a trained model efficiently in production, as a separate job from training the model in the first place.
  - The discipline exists because serving models at real scale surfaces constraints training never does — latency a user actually feels, cost per request that compounds across millions of calls, and GPU capacity that is genuinely scarce.
  - Training is a single project with a defined end; serving a model in production never finishes, which is why inference engineering looks more like running infrastructure than running an experiment.
check_questions:
  - q: Why is running a model in production a genuinely different job from training it?
    a: Training is a bounded project — it runs, finishes, and produces a checkpoint. Serving that checkpoint to real users runs continuously, under a latency budget the user actually feels, at a cost that compounds with every single request rather than being paid once.
  - q: Why might inference, not training, be the larger cost over a model's lifetime?
    a: Training happens once, however expensive. A popular model is called millions or billions of times afterward, and even a small per-request cost multiplied by that volume routinely exceeds what training cost in the first place.
deeper:
  - label: "Charles Frye (Modal) — What every AI engineer needs to know about GPUs"
    url: https://www.youtube.com/watch?v=y-UGrYbJsJk
related_formulas: []
---

## Serving is a different job from training

This roadmap's companion, Upshard's Machine Learning roadmap, ends with a trained
model. Inference engineering starts exactly there and asks a different question:
how do you run that trained model so that real users get fast, affordable, reliable
answers from it, at whatever volume the product actually needs? Training is research
and optimisation against a dataset. Inference engineering is systems work against a
live, continuously-arriving stream of real requests — closer in spirit to running a
database or a web service than to training a model.

## Why this became its own discipline

A few years ago, "serving a model" meant loading it and calling it — a small enough
problem that nobody needed a dedicated role for it. Three pressures changed that.
**Latency became a product requirement, not an afterthought**: a chat interface that
takes ten seconds to start responding is a failed product regardless of how accurate
the underlying model is. **Cost became a line item that scales with success**: a
feature that costs a fraction of a cent per call is free money at low volume and a
real budget line once it serves millions of requests a day. **GPU capacity became
scarce and expensive** in a way ordinary compute never was, which turned "how
efficiently are we using the hardware we have" into a question with real financial
weight behind it.

Put together, these three pressures created a job that did not clearly exist before:
someone whose entire focus is making a trained model fast, cheap and reliable to
call, without touching how it was trained at all.

## A concrete sense of the cost asymmetry

Consider a model that costs a notional ten thousand dollars to fine-tune for
a specific task — a real, one-time expense, paid once regardless of how the
model is subsequently used. If the resulting feature serves ten million
requests over its first year at even a fraction of a cent of inference cost
each, the serving bill alone can run into tens of thousands of dollars before
accounting for engineering time spent operating it, monitoring it, and
responding to incidents — and that cost recurs every single year the feature
stays in production, while the training cost was paid exactly once. This
asymmetry, repeated across every popular model in production, is the
concrete financial reality behind the abstract claim that inference
dominates a model's lifetime cost, and it is why an organisation running many
such features at scale eventually needs people whose entire job is making
that recurring cost as small as it can be without sacrificing the quality or
speed the product actually needs.

## Training ends; serving does not

A training run has a natural finish line: the model converges, or the budget runs
out, and you have a checkpoint. Production serving has no equivalent finish line. It
runs for as long as the product exists, under traffic that changes by time of day and
by season, against a latency budget that users notice the moment it is missed, and
against a bill that arrives whether or not anyone is watching it closely. This
difference in shape — a project with an end versus an operation with none — is why
inference engineering borrows more from running production infrastructure than from
anything in a typical machine learning course, and it is the lens every lesson in this
roadmap is written through: not "how do we make this model better," but "how do we
run this exact model well."
