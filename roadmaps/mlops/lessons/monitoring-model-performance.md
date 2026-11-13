---
title: Monitoring model performance
description: Why a model needs two separate kinds of monitoring — infra health and whether its predictions are still actually good — not just one.
roadmap: mlops
node: monitoring-model-performance
group: monitoring-governance
video:
  provider: youtube
  id: "D87eM6_N2GM"
  start: 0
  end: null
creator:
  name: Toronto Machine Learning Society (TMLS)
  channel_url: https://www.youtube.com/@tmls-ai
duration_seconds: 2226
level: intermediate
prerequisites: [deployment-strategies]
takeaways:
  - A model's infrastructure can be perfectly healthy — low latency, no errors, full throughput — while its predictions have quietly become wrong, because uptime metrics and prediction quality are measured by completely different signals.
  - Operational monitoring (latency, error rate, throughput, resource usage) answers "is the service running," while model monitoring (prediction distributions, confidence, proxy and ground-truth metrics) answers "is the model still right," and production systems need both.
  - Because true labels usually arrive late or never, most live model monitoring has to lean on proxies — prediction distributions, confidence scores, business outcomes — rather than waiting for ground truth to confirm a problem.
check_questions:
  - q: Why isn't a green dashboard of latency and error rate enough to know a model is still working?
    a: Those metrics only confirm the service is responding quickly and without crashing; they say nothing about whether the predictions coming back are still accurate, since a model can serve confidently wrong answers just as fast as correct ones.
  - q: Why does production model monitoring usually rely on proxy signals instead of directly measuring accuracy?
    a: Because the true labels needed to compute accuracy (the real outcome of a loan default, a fraud case, a click) often arrive with a delay of days or weeks, or never arrive at all, so teams substitute signals available immediately, like how confident the model is or how its prediction distribution has shifted.
  - q: What is an example of a proxy metric you could monitor in place of ground-truth accuracy?
    a: The distribution of the model's predicted probabilities, or its average confidence score, tracked over time — a sudden shift in that shape often signals a problem well before any labeled outcome would confirm it.
deeper:
  - label: "ML Model Performance Monitoring with WhyLabs — WhyLabs"
    url: "https://www.youtube.com/watch?v=8Tupxt0ciRI"
related_formulas: []
---

## Two questions that sound like one

"Is the model okay?" is really two separate questions wearing one sentence.
The first is operational: is the service that wraps the model up, fast, and
error-free? The second is statistical: are the predictions coming out of it
still good? Teams that have shipped plenty of conventional web services
often default to answering only the first question, because that's what
their existing tooling already measures — and that's exactly the gap that
causes quiet, expensive failures in ML systems. A model can sit at 50ms
p99 latency, zero 5xx errors, and full throughput for months while its
accuracy on real-world inputs has drifted so far that it's effectively
useless. Nothing in a standard infrastructure dashboard will show that,
because nothing about a wrong-but-confident prediction looks different, at
the infrastructure layer, from a right one.

## What operational monitoring actually covers

Operational (or infrastructure) monitoring for a model-serving system looks
like monitoring for any other service: request latency at various
percentiles, error rate, throughput, CPU/GPU/memory utilization, queue
depth if requests are batched, and uptime. This layer matters and is not
optional — a model that's technically accurate but times out under load or
throws exceptions on malformed input is still a failed system from the
user's perspective. But it's a necessary, not sufficient, layer. It answers
"can clients reach the model and get a response in reasonable time," and
stops there by design — these metrics are blind to what the response
actually contains.

## What model monitoring has to add

Model monitoring tracks the statistical behavior of the model's own
output and, where possible, how that output compares to reality. The
cleanest version of this is a ground-truth metric: compare predictions
against actual outcomes once they're known, and compute accuracy,
precision/recall, AUC, or whatever the task calls for, continuously, on a
rolling window. The catch is timing. In fraud detection, the true label
(was this transaction actually fraudulent) might arrive the next day after
a chargeback. In a recommendation system, you may never get a clean label
for "was this a good recommendation" at all. In a loan-default model, the
label might not resolve for months or years.

Because of that lag, most real monitoring setups lean heavily on proxy
signals that are available immediately: the distribution of predicted
classes or probabilities, the model's average confidence, the rate at which
predictions fall near a decision threshold (a sign of growing uncertainty),
and business-level outcomes like conversion rate or support-ticket volume
that correlate with model quality even without a direct label. None of
these proves the model is wrong on their own, but a sudden shift in any of
them — say, a classifier that used to predict the positive class 8% of the
time suddenly predicting it 35% of the time — is a strong, early signal
that something changed, worth investigating well before a lagging
ground-truth metric would confirm it.

## Building both layers into one picture

In practice, both kinds of monitoring should sit on the same dashboard,
because the question "what's wrong" usually requires correlating them. A
spike in latency that coincides with a shift in prediction distribution
might mean a batch of malformed or out-of-domain inputs just arrived. A
model whose accuracy quietly degrades with infrastructure metrics
completely flat points toward a data problem rather than a service
problem, which changes who gets paged and what they do next. Treating
these as one monitoring practice, rather than two separate dashboards
owned by two separate teams, is what turns monitoring into a tool for
diagnosis instead of just a tool for alerting.

## Why this is the right place to start

This lesson deliberately stays at the level of "what to watch and why the
split matters," because the two biggest specific failure modes behind
degrading model monitoring — drift in the input data and drift in the
real-world relationship the model learned — are substantial enough to earn
their own lesson next. Everything that follows in this closing part of the
roadmap, from diagnosing drift to explaining individual predictions to
responding to incidents, assumes this operational-plus-model monitoring
foundation is already in place. Without it, every later discipline is
reacting to symptoms it can't actually see.
