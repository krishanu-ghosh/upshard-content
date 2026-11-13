---
title: Data pipelines for ML
description: Building reliable pipelines that feed consistent data into training and inference, and what goes wrong when the two pull from separate paths.
roadmap: mlops
node: data-pipelines-for-ml
group: data-foundations
video:
  provider: youtube
  id: "s1DyAppdNmQ"
  start: 0
  end: null
creator:
  name: Tecton
  channel_url: https://www.youtube.com/@tecton8241
duration_seconds: 1879
level: beginner
prerequisites: [data-versioning]
takeaways:
  - A data pipeline is the sequence of extract, validate, transform, and load steps that turns raw source data into the exact inputs a model consumes, run the same way every time.
  - Training and serving pipelines frequently diverge because they're built separately for different constraints — batch throughput versus request latency — even when the engineers intend them to compute the same thing.
  - A pipeline without validation steps doesn't fail loudly when upstream data changes; it silently feeds the model different data, which is far more dangerous than a crash.
check_questions:
  - q: What are the core stages a data pipeline for ML typically performs?
    a: Extracting raw data from its source, validating it against expected shape and ranges, transforming it into model-ready features, and loading the result to wherever training or serving reads from.
  - q: Why do training and serving pipelines tend to drift apart even when built by the same team with good intentions?
    a: Because they're usually built under different constraints — training optimizes for batch throughput over historical data while serving optimizes for low-latency response to a single request — so teams write two separate implementations that quietly diverge as each one changes over time.
  - q: Why is a pipeline with no validation step more dangerous than one that crashes on bad input?
    a: Because it keeps running and silently feeds the model corrupted, shifted, or malformed data instead of failing visibly, so the damage shows up later as degraded predictions with no obvious cause.
deeper: []
related_formulas: []
---

## What a pipeline actually is

A data pipeline for ML is the automated sequence that turns whatever raw
data exists — rows in a production database, event logs, files dropped in
object storage — into the exact tensors or feature vectors a model
actually consumes. Conceptually it has four stages that repeat on every
run: **extract** (pull raw records from the source system), **validate**
(check the data looks like what's expected before trusting it),
**transform** (clean, join, aggregate, and engineer features from it),
and **load** (write the result somewhere training or serving can read it).
None of this is exotic — it's the same shape as any ETL pipeline — but ML
pipelines carry an extra requirement ordinary data pipelines don't: the
output has to match, feature for feature, whatever the model was trained
to expect, every single time the pipeline runs, whether that's once a day
for a batch retrain or once per millisecond for a live prediction request.

## Why pipelines break quietly

The most damaging pipeline failures aren't crashes — crashes get noticed
and fixed fast. The damaging ones are silent shape changes in the data
that the pipeline keeps processing without complaint. Say an upstream
service that used to send a `country` field as a two-letter code starts
sending full country names after an unrelated update elsewhere in the
company. A pipeline with no validation step doesn't error out; it just
feeds a categorical feature encoder values it's never seen, and depending
on how that encoder handles unknowns, the model either gets garbage for
that feature or the encoder quietly maps everything new to a default
bucket. Either way, prediction quality degrades for exactly the users
affected by that change, and nothing in the system raised an alarm,
because nothing was technically broken — the pipeline ran, produced
output, and loaded it, same as always. This is why validation is treated
as a first-class pipeline stage and not an afterthought: schema checks
(are the expected columns present, with the expected types), range checks
(is this numeric feature still inside its historical bounds), and
null-rate checks (did a normally-populated field suddenly go mostly empty)
catch exactly this class of problem at the moment it enters the system,
rather than weeks later when someone notices accuracy has drifted.

## Where training and serving split — and why that's dangerous

The deeper structural problem is that training and serving pipelines
almost never start life as the same pipeline. Training runs over
historical data in bulk: a nightly job can take twenty minutes to compute
features for millions of rows, written in whatever language or framework
is convenient for batch analytics — commonly Python with pandas or Spark.
Serving runs under a latency budget measured in milliseconds, for one
request at a time, often written in a different language chosen for speed
rather than analytical convenience. Because the constraints are so
different, it's natural — almost inevitable — for a team to end up with
two separate implementations of "compute this feature," one for the batch
path and one for the real-time path. They start out logically equivalent.
Then one gets patched for an edge case, and the other doesn't. A "days
since signup" feature handles a null signup date as zero in one pipeline
and as a sentinel value of negative one in the other, and now the model
is silently seeing two different encodings of "no signup date yet"
depending on whether it's being trained or queried live. This exact split
is what the next lesson's fix — a shared feature store both paths read
from — exists to close; a well-built pipeline is the piece that makes
that shared source of truth possible in the first place.

## Why this discipline compounds

Get pipelines right and every downstream practice gets easier: a
reproducible pipeline run, paired with a versioned dataset, means a
training run can be rebuilt exactly; validated input means degraded model
performance can be ruled in or out as a data problem quickly instead of
through days of guessing; and a pipeline designed with training and
serving in mind from the start avoids the skew that would otherwise have
to be discovered the hard way, in production, after it's already cost
accuracy. Pipelines are unglamorous infrastructure, but almost every
other MLOps practice in this roadmap — versioning, feature stores,
monitoring, retraining — assumes a pipeline underneath it that behaves
the same way today as it will next month.
