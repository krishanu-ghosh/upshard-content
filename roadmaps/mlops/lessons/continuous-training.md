---
title: Continuous training
description: Automatically retraining when data arrives or performance drops — and the guardrails that stop an automated retrain from shipping a worse model.
roadmap: mlops
node: continuous-training
group: ci-cd
video:
  provider: youtube
  id: "giNPl2Uo0xc"
  start: 0
  end: null
creator:
  name: Toronto Machine Learning Society (TMLS)
  channel_url: https://www.youtube.com/@tmls-ai
duration_seconds: 2684
level: advanced
prerequisites: [testing-ml-systems]
takeaways:
  - Continuous training replaces a human-scheduled retrain with a system that triggers on a signal — new data volume, elapsed time, or measured performance degradation — because real-world drift doesn't follow a calendar.
  - A retrain trigger without a quality gate is dangerous, not helpful, because it automates the decision to retrain but not the decision to deploy, and those have to be separated.
  - The guardrail that makes continuous training safe is comparing the newly trained model against the currently deployed model on the same evaluation set before promoting it, so a worse model never auto-replaces a better one.
check_questions:
  - q: What are the two main kinds of triggers for continuous training, and what does each respond to?
    a: A schedule- or volume-based trigger fires on elapsed time or accumulated new data, while a performance-based trigger fires when a monitored metric, like live accuracy or prediction drift, crosses a defined threshold.
  - q: Why is a retrain trigger alone not sufficient to automate safely?
    a: Because triggering a retrain only produces a new model candidate, and without a separate quality gate that compares it against the current production model, the system has no way to stop a worse candidate from being promoted automatically.
  - q: Why must the challenger model be evaluated on the same held-out data as the current production model, not just on new data?
    a: Because comparing on a shared, fixed evaluation set isolates whether the new model is actually better, while evaluating each model on different data could make a worse model look better purely from a difference in what was being measured.
deeper:
  - label: Incremental And Continuous Model Training — Krish Naik
    url: https://www.youtube.com/watch?v=T4y25jc5NyM
related_formulas: []
---

## Retraining on a trigger instead of a calendar

Manual retraining looks like this: someone on the team notices the model
feels stale, or a quarterly calendar reminder fires, and they kick off a
training job by hand. That works until the gap between "the world changed"
and "someone noticed" becomes the thing costing the business money. A
fraud model trained on last year's attack patterns doesn't wait for a
human's schedule to become less effective — it degrades the moment fraud
tactics shift, which can happen in days. Continuous training replaces the
calendar with a trigger condition evaluated automatically: retrain when
enough new labeled data has accumulated, retrain on a fixed cadence tight
enough to track known drift, or — the more sophisticated version —
retrain when a live performance metric crosses a threshold, such as
prediction confidence dropping, the distribution of input features
shifting measurably from the training distribution, or downstream
accuracy (once ground truth catches up) falling below an acceptable
floor. The trigger doesn't have to be exotic; it has to be *measured*,
because "the model has probably gotten worse" is a guess and "this metric
crossed this threshold" is a fact a pipeline can act on.

## Why a trigger alone is the dangerous half

Here's the part that makes continuous training genuinely harder than it
sounds: automating when a retrain starts is the easy half. A trigger
firing just produces a new candidate model — it hasn't decided whether
that candidate should replace what's currently serving traffic. If the
pipeline promotes every freshly trained model automatically, it has
built a system that can silently make things worse at machine speed. A
retrain triggered by a sudden traffic anomaly, a batch of mislabeled
data, or a data pipeline bug that just happened to coincide with the
trigger condition can produce a model that's confidently worse than the
one it's replacing — and because retraining "succeeding" only means the
job completed without crashing (the same failure mode covered in the
CI lesson), nothing stops a bad candidate from going out the door unless
something explicitly checks its quality first.

## The guardrail: compare, don't just replace

The fix is to treat every retrained model as a *challenger*, not an
automatic *replacement*. Before a continuously trained model goes live,
it gets evaluated against the currently deployed model — the champion —
on the same fixed, held-out evaluation set, using the same metrics the
team already trusts. Same test set matters specifically: if the
challenger were evaluated on newer data and the champion's historical
score were used for comparison, an apparent improvement could just be an
easier test set, not a better model. Only if the challenger clears both
an absolute bar (it's good enough on its own terms) and a relative bar
(it's at least as good as what's running now, usually with a safety
margin) does it get promoted — often first to a small slice of traffic
via a canary or shadow deployment, with a human or an automated rollback
rule watching live outcomes, before it fully replaces the champion. This
is the same instinct as a deployment approval gate in ordinary software
CD, applied to a decision that in ML carries extra risk: nothing about
"the training job finished" implies "this model is actually better."

## What's really being automated

It's worth being precise about what continuous training buys you: it
automates *noticing that retraining is warranted* and *producing a
candidate*, not the judgment call of whether to ship it. Teams that get
this wrong usually do so by conflating the two — wiring the trigger
directly to production and treating "it retrained" as success. Teams
that get it right keep retraining fast and frequent while keeping
promotion gated, expensive to fail, and comparative rather than absolute.
That separation is exactly what lets an ML team retrain as often as the
real world demands without taking on the risk of shipping a worse model
every time it does. And once candidates are reliably produced and
safely gated, the remaining piece is mechanical rather than judgment-
based: getting a validated model artifact from "passed its evaluation
gate" to "actually packaged and deployable" without a manual handoff,
which is where continuous delivery for models picks up next.
