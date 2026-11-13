---
title: Feature stores
description: Centralizing engineered features so training and serving read the exact same logic and values, closing one of the top causes of training/serving skew.
roadmap: mlops
node: feature-stores
group: data-foundations
video:
  provider: youtube
  id: "DESBDukN9gw"
  start: 0
  end: null
creator:
  name: Hopsworks
  channel_url: https://www.youtube.com/@hopsworks
duration_seconds: 263
level: beginner
prerequisites: [data-pipelines-for-ml]
takeaways:
  - A feature store separates feature computation from feature consumption, so a feature is defined once and read identically by both training and serving instead of being reimplemented twice.
  - Feature stores typically split storage into an offline store for bulk historical training data and an online store for low-latency single-record lookups at prediction time, fed by the same underlying feature definitions.
  - Point-in-time correctness — fetching feature values exactly as they existed at a past moment, not their current values — is what makes historical training data trustworthy instead of leaking future information into the past.
check_questions:
  - q: What specific problem does a feature store solve that a well-built pipeline alone doesn't fully fix?
    a: It removes the need for two separate implementations of feature logic for training and serving by giving both the exact same computed feature values from a single shared source.
  - q: Why do feature stores split storage into an "offline" and an "online" store?
    a: Training needs fast bulk access to large volumes of historical feature data while serving needs very low-latency lookup of a single record's current features, and no single storage system is optimized for both access patterns at once.
  - q: What is point-in-time correctness, and why does it matter for training data?
    a: It means retrieving a feature's value exactly as it was at a specific past timestamp rather than its current value, which matters because using a feature's present-day value for a historical training example would leak information the model couldn't have actually had at prediction time.
deeper:
  - label: "Understanding the Feature Store: Literal, Physical & Virtual Explained — neptune_ai"
    url: "https://www.youtube.com/watch?v=hGsTvKnbpjg"
related_formulas: []
---

## The problem one more time, more precisely

The previous lesson in this group ended on a specific failure: training
and serving pipelines computing the "same" feature two different ways
because they were built separately, under separate constraints, and
slowly drifted apart. A feature store is the infrastructure built
specifically to make that drift structurally impossible rather than
something you have to keep policing by hand. The core idea is simple to
state: define a feature's computation logic exactly once, and make both
the training pipeline and the serving system pull their values from that
single definition, instead of each maintaining — and inevitably diverging
in — its own copy of the logic.

## Two storage systems, one source of truth

A feature store is usually split into two physical stores that serve very
different access patterns from the same underlying feature definitions.
The **offline store** holds the full history of feature values over time,
optimized for the kind of access training needs: pulling millions of rows
covering months of history in a single bulk query, to assemble a training
dataset. The **online store** holds only the current, latest value of
each feature per entity — one customer, one product, one session — and is
optimized for the opposite access pattern: a single-key lookup returning
in single-digit milliseconds, because a live prediction request can't
wait on anything slower. Both are populated by the same feature
computation pipeline, so "average purchase amount over the last 30 days"
for a given customer is computed by one piece of logic, written to both
stores, and read by training from one side and by serving from the other
— never recomputed separately by each consumer. This is the direct fix
for the skew problem: there is no second implementation left to drift.

## Point-in-time correctness, and why it's the hard part

The subtle piece a feature store has to get right — and the part that's
easy to get wrong if you build this yourself — is point-in-time
correctness. When you assemble a training dataset, each training example
is anchored to a specific moment in the past: "this customer, as they
looked right before this specific purchase on March 3rd." If you naively
join that example against a feature table using today's current feature
values, you leak the future into the past — the model trains on
information (like "total purchases this customer ever made," computed as
of today) that didn't exist yet at the moment the real prediction would
have been made. A model trained this way looks great in offline
evaluation and then underperforms badly in production, because production
never gets to see the future. A feature store solves this by keeping
timestamped history in the offline store and resolving each training
example against the feature values as they actually stood at that
example's specific timestamp — not the latest value, the value that was
true then. Getting this join right across millions of rows and many
features is exactly the kind of fiddly, easy-to-get-subtly-wrong
infrastructure work that justifies a dedicated system instead of ad hoc
SQL someone writes once and never revisits.

## A concrete walk-through

Say you're predicting loan default risk, and one feature is "number of
late payments in the last 90 days." Without a feature store, the training
pipeline computes this with a batch SQL query over a payments table, while
the serving system maintains a running counter updated as payments come
in. The SQL query's window might be calendar days; the running counter
might be rolling 90*24 hours, and the two can disagree right at the
boundary of a payment that's 89.5 days old. With a feature store, both
sides call the same feature definition — one function, one window logic
— and the online store simply holds that function's most recent computed
output, kept fresh by the same pipeline that also backfills the offline
store's history. There's no second implementation anywhere for the
boundary case to hide in.

## Why this matters going forward

Feature stores are the concrete, buildable answer to a problem the last
two lessons described at increasing levels of specificity: data drifts,
pipelines diverge, and models silently see something different from what
they were trained on. A feature store doesn't eliminate the need for good
pipelines or versioned data — it sits on top of both — but it closes the
single most common and most expensive gap between training and serving by
removing the duplicate implementation that gap depends on. The next
lesson, on data lineage, picks up the remaining question this doesn't
answer: once a feature's value is in front of you, how do you trace
exactly where it came from and what touched it along the way.
