---
title: Data and concept drift
description: Why incoming data or the real relationship a model learned can shift after training, silently degrading accuracy a healthy service won't show.
roadmap: mlops
node: data-and-concept-drift
group: monitoring-governance
video:
  provider: youtube
  id: "QJTRNxUxmuc"
  start: 0
  end: null
creator:
  name: 1littlecoder
  channel_url: https://www.youtube.com/@1littlecoder
duration_seconds: 625
level: intermediate
prerequisites: [monitoring-model-performance]
takeaways:
  - Data drift is a shift in the distribution of a model's inputs since training, while concept drift is a shift in the underlying relationship between inputs and the correct output — they require different diagnosis and different fixes.
  - Both kinds of drift degrade accuracy without touching any infrastructure metric, because the model keeps serving fast, confident predictions that are simply built on assumptions that no longer hold.
  - Detecting drift usually means comparing a statistical summary of recent input data (or recent predictions) against a stored baseline from training, using tests built for exactly that comparison, rather than waiting for labeled outcomes to prove something went wrong.
check_questions:
  - q: What's the difference between data drift and concept drift?
    a: Data drift is a change in the distribution of the inputs the model receives, while concept drift is a change in the true relationship between those inputs and the correct output, meaning the same input can start to warrant a different answer than it did at training time.
  - q: Why can a model experiencing drift still look perfectly healthy on a standard infrastructure dashboard?
    a: Because drift affects the statistical validity of predictions, not the service's ability to compute and return them quickly and without errors, so latency, error rate, and throughput all stay normal while accuracy quietly falls.
  - q: Why is concept drift generally harder to detect than data drift?
    a: Data drift can often be caught by comparing the distribution of inputs alone, with no labels required, while concept drift only shows up once you compare predictions against actual outcomes, which are frequently delayed or sparse in production.
deeper:
  - label: "ML Drift: Identifying Issues Before You Have a Problem — Fiddler AI"
    url: "https://www.youtube.com/watch?v=uOG685WFO00"
related_formulas: []
---

## The assumption every model is secretly making

Every trained model carries a hidden assumption: the data it sees in
production will look statistically like the data it was trained on, and
the relationship between inputs and correct outputs will stay the same.
That assumption is called the i.i.d. assumption in textbooks, and it is
false, eventually, for essentially every model deployed for more than a
few months. The world the model was trained to describe keeps moving —
customer behavior changes, a competitor launches a product, a pandemic
reroutes an entire economy, a sensor gets recalibrated — and the model has
no way to notice on its own. This lesson is about naming and detecting the
two distinct ways that assumption breaks.

## Data drift: the inputs themselves change

**Data drift** (also called covariate shift) is a change in the
distribution of the model's input features, independent of whether the
input-output relationship itself has changed. A credit model trained
mostly on applicants aged 25-45 that starts receiving a surge of
applicants aged 55-70 is experiencing data drift — the demographic mix of
`age` has shifted. Importantly, the model's *learned rule* might still be
perfectly correct for every individual applicant; the problem is that it's
now operating heavily in a region of the input space it saw rarely during
training, where its predictions are much less reliable, simply because it
had little data there to learn from. Data drift is detectable without any
labels at all — it's purely a property of the inputs — using statistical
tests that compare a recent window of production data against a stored
training-time baseline: population stability index, Kolmogorov-Smirnov
tests for continuous features, or simple divergence measures like KL or
Jensen-Shannon divergence on the feature distributions.

## Concept drift: the relationship itself changes

**Concept drift** is a different and often more serious problem: the true
mapping from input to correct output has changed, even if the inputs look
statistically identical to training. A spam classifier trained on old
spam patterns faces concept drift when spammers adopt new tactics
specifically designed to look like the legitimate email the model was
trained to let through — the words in the email (the inputs) might fall
in a perfectly familiar range, but what those words now *mean* about
whether the email is spam has changed. A fraud model faces concept drift
when fraud rings change their playbook. A demand-forecasting model faces
concept drift when a pandemic changes what "normal" purchasing behavior
even is. Because concept drift is about the relationship rather than the
inputs, detecting it generally does require comparing predictions against
ground truth, which is exactly why it tends to surface later and cause
more damage before anyone notices — it's hiding behind the very labels
that are slowest to arrive.

## Why conflating the two leads you astray

Treating drift as one undifferentiated problem leads to the wrong fix.
Data drift is often addressed by retraining on more representative recent
data, or by flagging predictions made far outside the training
distribution as lower-confidence. Concept drift can't be fixed by simply
adding more recent data of the old kind — the *target* has moved, so the
retraining has to happen with freshly labeled examples that reflect the
new relationship, and in severe cases (a fundamentally new fraud pattern)
even the feature set or model architecture may need to change. Diagnosing
which kind of drift is occurring, by checking input distributions first
and ground-truth performance second, is what tells a team whether a quick
retrain will fix things or whether they're facing a deeper redesign.

## Carrying this forward

Drift detection is the mechanism that turns "the model's predictions are
silently getting worse," the failure mode the previous lesson described,
into something you can actually see and act on before it does damage. It
feeds directly into the governance practices coming next: an audit trail
that can show when drift was detected and what was done about it is a
core piece of what regulators and internal reviewers expect from a
production ML system, and it's the first signal most incident responses
to a misbehaving model actually start from.
