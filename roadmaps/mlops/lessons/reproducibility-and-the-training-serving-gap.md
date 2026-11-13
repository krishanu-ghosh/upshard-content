---
title: Reproducibility and the training/serving gap
description: Why a model that trained perfectly can behave differently in production — and the concrete mismatches between training and serving code that cause it.
roadmap: mlops
node: reproducibility-and-the-training-serving-gap
group: foundations
video:
  provider: youtube
  id: "pqe-HB7ZcUI"
  start: 0
  end: null
creator:
  name: Cassie Kozyrkov
  channel_url: https://www.youtube.com/@kozyrkov
duration_seconds: 185
level: beginner
prerequisites: [the-ml-lifecycle]
takeaways:
  - Training/serving skew happens when the features a model sees in production differ, even subtly, from the features it saw during training — and the model has no way to flag that mismatch itself.
  - The most common cause isn't a dramatic bug but two separate code paths — one for batch training, one for live serving — that compute "the same" feature in slightly different ways.
  - Reproducibility means pinning everything that can silently change a result — code, data, library versions, and the environment the code runs in, not just the model file itself.
check_questions:
  - q: What is training/serving skew?
    a: A mismatch between the data or features a model was trained on and the data or features it actually receives at prediction time, which degrades its real-world accuracy even though nothing about the model itself changed.
  - q: Why is having two separate code paths for computing features — one offline for training, one online for serving — a common source of skew?
    a: Because even logically "equivalent" code written twice tends to drift apart over time as one path gets updated and the other doesn't, so the two paths can compute subtly different values for what's supposed to be the same feature.
  - q: Besides the feature code itself, what else needs to be pinned to make a model's behavior reproducible between training and serving?
    a: The exact versions of libraries and dependencies and the surrounding runtime environment, since the same code can behave differently under a different library version or environment configuration.
deeper:
  - label: "Machine Learning Platform for Online Prediction and Continual Learning — Chip Huyen, apply() Conference 2022"
    url: https://www.youtube.com/watch?v=P5uBWGqzogs
related_formulas: []
---

## A model that "worked" can still be wrong in production

A model is trained, it hits a strong accuracy number against a held-out
test set, everyone is satisfied, and it ships. Weeks later, someone
notices the live predictions look off — not broken, just worse than the
evaluation numbers promised. Nothing about the model's weights changed.
What changed is more subtle: the data the model sees in production isn't
quite the data it was trained on, even though it's nominally "the same"
features. This is training/serving skew, and it's one of the most common
and hardest-to-detect failure modes in deployed ML systems, precisely
because the model itself is innocent — it's doing exactly what it learned
to do, on inputs that no longer quite match what it learned from.

## Where the mismatch actually comes from

The textbook cause is having two separate implementations of feature
computation: an offline pipeline (often Python with pandas or Spark) that
builds the training dataset, and a separate online service (often a
different language, for latency reasons) that computes features at
request time to feed the live model. Say a "days since last purchase"
feature is computed in training with pandas, where a missing last-purchase
date becomes `NaN` and gets filled with a sentinel value like `-1`. The
production feature service, written separately in a lower-latency
language, computes the same logical feature but treats a missing date as
`0` instead, because that was the natural default for whoever wrote that
code six months later. Nobody decided this inconsistency; it just
accumulated because the two code paths were never required to produce
identical output, and nothing tested that they did. The model, trained
on `-1` meaning "no purchase history," now silently receives `0` for the
same real-world condition in production — a value it associates with
"purchased very recently." Every new or lapsed customer gets scored as if
they just bought something, and nobody wrote a line of broken code to
cause it.

A second common source is a library version mismatch. A text feature
built with one version of a tokenizer library during training can
tokenize slightly differently under a newer version installed in the
serving environment — different handling of punctuation, different
default lowercasing — producing a different token sequence for what looks
like the same input text. The model was never retrained on this new
tokenization; it's silently seeing out-of-distribution input it was never
designed for.

A third is environment drift more broadly: the training machine and the
serving machine can differ in OS-level numeric libraries, hardware
(CPU vs. GPU floating-point behavior), or even locale settings that
affect how dates or numbers get parsed. Individually tiny, these
differences compound across many features into a meaningfully different
input distribution.

## Why this is hard to catch

Training/serving skew is insidious because standard validation doesn't
catch it. The model passed evaluation — on the training pipeline's version
of the features. The serving infrastructure passed its load tests —
correctly, on whatever it computes, without reference to what training
assumed. Each side looks correct in isolation; the mismatch only exists
in the gap between them, which is exactly the gap that's easiest for
standard testing to miss because no single test owns it.

## What reproducibility actually requires

Solving this isn't about writing more careful code once — it's about
removing the opportunity for two implementations to diverge at all. The
most reliable fix is a shared feature pipeline: the same feature-computing
code (or a tightly shared definition, as in a feature store) used for both
training and serving, so there's structurally only one way to compute
"days since last purchase," not two that are supposed to agree. Where
that's not fully possible, reproducibility means pinning every version
that can affect output — library versions, environment configuration,
even random seeds — and treating that pinned environment as part of what
gets deployed, not just the model weights.

## Why this closes the foundations

This lesson is the sharp, concrete version of the gap the last two
lessons described more abstractly: a model is a function of data and
environment as much as of code, and nothing about "the model passed
evaluation" guarantees it will see the inputs it expects once it's live.
Every practice covered later in this roadmap — feature stores,
versioned pipelines, shadow deployments, monitoring that compares live
feature distributions against training distributions — exists to close
exactly this gap before it costs you silently, in production, for weeks
before anyone notices.
