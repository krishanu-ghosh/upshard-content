---
title: Model serving patterns
description: Batch, online, and streaming inference are different architectures with different failure modes, not interchangeable ways to run the same model.
roadmap: mlops
node: model-serving-patterns
group: serving-deployment
video:
  provider: youtube
  id: "W3hKjXg7fXM"
  start: 0
  end: 1260
creator:
  name: The Full Stack
  channel_url: https://www.youtube.com/@The_Full_Stack
duration_seconds: 3448
level: advanced
prerequisites: [continuous-delivery-for-models]
takeaways:
  - Batch, online, and streaming inference are distinct architectures with different latency, cost, and freshness tradeoffs, not just configuration options on the same deployment.
  - The right serving pattern is determined by how fresh a prediction needs to be when it's consumed, not by which pattern is easiest to build first.
  - Picking the wrong pattern doesn't just cost performance — it can make a feature impossible to ship or silently wrong in production.
check_questions:
  - q: What question should drive the choice between batch and online inference?
    a: How fresh does the prediction need to be at the moment it's consumed, relative to how often the underlying input data changes?
  - q: Why can batch inference still be the right choice even though it serves "stale" predictions?
    a: Because for many use cases the input data doesn't change fast enough for freshness to matter, and batch is far cheaper and simpler to operate than an always-on prediction service.
  - q: What's the core architectural cost that online inference pays to get low-latency, per-request predictions?
    a: It requires a continuously running, scaled, and monitored service that must respond fast enough for a user to be waiting on it, unlike batch jobs that can run on a schedule with no one waiting.
deeper:
  - label: "Principles Of Good ML Systems Design — Chip Huyen, Toronto Machine Learning Society (TMLS)"
    url: "https://www.youtube.com/watch?v=t9cwk3zB7N0"
related_formulas: []
---

## Three ways to get a prediction out of a model

A trained model is just a function sitting on disk until something decides
*when* it runs and *who's waiting* for the answer. That decision is the
serving pattern, and there are three fundamentally different ones: batch,
online (real-time), and streaming. They aren't three implementations of the
same idea — they trade off latency, cost, infrastructure complexity, and
prediction freshness against each other in ways that make the "wrong"
choice for a given use case not just suboptimal but sometimes unworkable.

**Batch inference** runs the model over a large set of inputs on a
schedule — nightly, hourly, whatever cadence fits — and writes the
predictions somewhere for later lookup. Nobody is waiting on the model
while it runs. A churn-risk score recomputed every night for every
customer, written to a table that a dashboard reads from, is batch
inference. **Online inference** runs the model synchronously, on demand,
in response to a single request, with a human or another service waiting
for the response. A fraud check that has to return a yes/no before a
credit card transaction completes is online inference — it cannot be
precomputed because the input (this specific transaction, right now)
doesn't exist until the request arrives. **Streaming inference** sits
between them: the model runs continuously against a flow of events as they
arrive, typically asynchronously, updating predictions incrementally
rather than on a fixed schedule or a single blocking request. A model
scoring clicks on a live event stream to update a recommendation ranking
is streaming inference.

## Why the choice is about freshness, not difficulty

It's tempting to default to whichever pattern is easiest to build, but the
pattern is actually determined by a property of the problem: how fast does
the input change relative to how fast you need an answer? If a customer's
churn risk realistically shifts over days or weeks, computing it every
night is not a compromise — it's correctly matched to the pace of the
underlying signal, and it's dramatically cheaper and simpler than standing
up an always-on service for no real benefit. Batch jobs can run on cheap,
non-urgent infrastructure, can be retried freely if they fail, and don't
need to meet a strict response-time budget.

Online inference exists because some inputs genuinely don't exist until
request time. You cannot precompute a fraud score for a transaction that
hasn't happened yet, or a search-ranking response for a query no one has
typed yet. That forces a different kind of system: one that's always
running, horizontally scaled to handle request volume, monitored for
latency percentiles rather than job completion, and built to degrade
gracefully (a timeout, a fallback) rather than simply fail a batch job and
retry later. This is a meaningfully heavier operational commitment than
batch, which is exactly why it should be reserved for cases where request-time
computation is actually required.

Streaming inference earns its complexity when neither extreme fits: the
input is continuously arriving (not a single discrete request) but the use
case can't tolerate waiting for the next scheduled batch run. A
recommendation system that should reflect what a user clicked thirty
seconds ago, not what they clicked as of last night's batch job, needs
something that updates continuously. Streaming systems bring their own
cost — message queues, stateful stream processors, careful handling of
out-of-order or late-arriving events — that batch and even online systems
don't have to deal with.

## What goes wrong with the wrong pattern

Picking batch when you needed online inference doesn't just degrade
quality — it can make the feature not work at all, since the input
literally doesn't exist ahead of time. Picking online when batch would
have sufficed is a quieter failure: you've built and now have to operate a
24/7 scaled service, with its own on-call burden and latency SLAs, to serve
predictions that didn't need to be fresher than once a day. Streaming is
the pattern most often reached for prematurely, because "real-time" sounds
impressive, when the actual requirement was closer to "updated every few
minutes" — a gap that batch or a short-interval scheduled job would have
closed at a fraction of the operational cost.

## Carrying this forward

Every lesson after this one in this group — how to roll out a new model
version safely, how to handle real production load, how to run inference
on a constrained device — assumes you've already picked a serving pattern
and are now operating within it. Get this choice wrong and those later
concerns either don't apply the way you expect or get bolted onto an
architecture that was never suited to the problem. Getting it right starts
with one question, asked before any infrastructure gets built: how fresh
does this prediction actually need to be when someone consumes it?
