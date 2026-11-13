---
title: Taking Models to Production
description: What happens to a model after the notebook, and which of those steps a data scientist actually owns.
roadmap: ai-data-scientist
node: taking-models-to-production
group: shipping-insights
video:
  provider: youtube
  id: "tSiS15ubQFQ"
  start: 0
  end: null
creator:
  name: Thu Vu
  channel_url: https://www.youtube.com/@Thuvu5
duration_seconds: 792
level: intermediate
prerequisites: [model-interpretability-and-explainability]
takeaways:
  - A model that works in a notebook has to be wrapped in a serving layer, fed data through a pipeline that matches training-time preprocessing exactly, and monitored, none of which the notebook does for you.
  - "Training-serving skew — differences between how features were computed during training versus at prediction time — is one of the most common ways a good model quietly becomes a bad one in production."
  - Batch prediction and real-time prediction are different engineering problems with different latency, infrastructure, and failure-mode requirements, and the choice should follow from how the prediction gets used.
  - A data scientist's job is usually to define what "working correctly" means for a deployed model and to own the data and feature logic; the deployment plumbing itself is usually built and maintained by ML or platform engineers.
check_questions:
  - q: What is training-serving skew and why is it dangerous?
    a: "It's a mismatch between how a feature was computed when the model was trained and how it's computed at prediction time, and it's dangerous because the model keeps producing predictions with no error, just increasingly wrong ones."
  - q: Why does it matter whether a model serves predictions in batch or in real time?
    a: Batch prediction can tolerate slow, heavy computation run on a schedule, while real-time prediction has to return an answer within a user-facing latency budget, so the same model needs different infrastructure and different feature-computation code depending on which one it's used for.
  - q: What part of taking a model to production does a data scientist typically still own, even without being an MLOps engineer?
    a: Defining the success metric and acceptable failure modes for the deployed model, and making sure the feature computation used in production matches what the model was actually trained on.
deeper:
  - label: What is MLOps?
    url: https://www.youtube.com/watch?v=OejCJL2EC3k
related_formulas: []
---

## The notebook is not the product

A model that scores well in a notebook has cleared the easiest bar it will ever face. The notebook has clean, static, already-joined data; production has to pull that same data from live systems, on a schedule or on demand, without a human rerunning a cell when something looks off. Getting from "this model works on my test set" to "this model is making decisions inside a live product" means solving problems the notebook never asked you to solve: how predictions get requested, how the model's inputs get computed consistently every single time, how you'd know if the model started being wrong, and who gets paged when it does. None of that is deep learning theory. It's engineering, and it's worth understanding even if you never build it yourself.

## Training-serving skew: the silent failure mode

The single most common way a good model becomes a bad one after deployment is training-serving skew: a gap between how a feature was computed during training and how it's computed in production. A simple, realistic example. Suppose a churn model uses `days_since_last_purchase` as a feature, computed in training from a clean, already-deduplicated purchase table using `(today - max(purchase_date))`. In production, that same feature gets computed from a live events table that includes test transactions and duplicate webhook deliveries, and "today" is now a server timestamp in UTC instead of the analyst's local timezone used at training time. The model still runs. It still returns a probability between 0 and 1. It is just systematically wrong, because the number in the feature slot no longer means what it meant when the model learned from it. There's no error message for this — only a slow drift in accuracy that someone has to notice, which is exactly why monitoring deployed models (not just deploying them) is part of the job.

Avoiding this requires the feature computation logic to be shared, not reimplemented, between training and serving. In practice this means things like a shared feature store, or a feature-computation function that's unit tested and imported by both the training pipeline and the serving pipeline, rather than hand-copied from a notebook into a service.

## Batch versus real time, and why the choice isn't free

Models get served in one of two shapes, and the shape determines a lot of the engineering around it.

**Batch prediction** runs the model on a large set of rows on a schedule, say nightly, and writes the predictions to a table that downstream systems read from. This is the right shape when predictions don't need to reflect something that happened in the last five minutes: a monthly churn-risk score, a weekly demand forecast per store. Batch is forgiving. It can use slower, heavier feature computation, and if a run fails, you can usually rerun it before anyone notices.

**Real-time prediction** serves a prediction on demand, typically through an API call triggered by a live event — a fraud score computed the instant a transaction is submitted, a recommendation generated when a user loads a page. This is far less forgiving. The service has a latency budget, often under a few hundred milliseconds, which constrains how features can be computed: you usually can't run a complex SQL join against a warehouse in that window, so real-time features often come from a pre-computed cache or a stream processor instead. Real-time systems also need to handle the request volume reliably, with fallbacks for when the model service is slow or down, because unlike a batch job, a failure here is visible to a user immediately.

Choosing between them isn't a technical preference; it follows directly from how the prediction will be used. A data scientist who understands this distinction can scope a project realistically instead of promising a "real-time personalization model" that actually only needs to update once a day.

## Where the data scientist's job ends

None of this means a data scientist needs to become a deployment engineer. In most organizations, the actual serving infrastructure, container orchestration, API scaffolding, and monitoring dashboards are built and operated by ML engineers or a platform team. What stays squarely in the data scientist's lane is defining what correct behavior looks like: the metric that should be tracked post-deployment, the range of inputs the model was validated on (so engineering knows what's out of scope), and the feature logic itself, which needs to be specified precisely enough that it can be reimplemented identically outside the notebook. Knowing the shape of the production problem is what lets a data scientist hand off a model that survives contact with production, rather than one that works until the first retraining cycle quietly breaks it.

This lesson is an orientation, not the full picture. Upshard's dedicated `mlops` roadmap covers the engineering depth of this topic in full, including deployment patterns, monitoring architecture, and retraining pipelines, for anyone who wants to go further than a data scientist typically needs to.
