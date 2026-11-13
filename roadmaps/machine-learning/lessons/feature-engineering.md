---
title: Feature engineering
description: Creating new input variables from raw data that make patterns easier to learn — transformations, interactions, aggregations and date decomposition.
roadmap: machine-learning
node: feature-engineering
group: working-with-data
video:
  provider: youtube
  id: "pYVScuY-GPk"
  start: 0
  end: null
creator:
  name: codebasics
  channel_url: https://www.youtube.com/@codebasics
duration_seconds: 311
level: intermediate
prerequisites: [data-cleaning]
takeaways:
  - A feature is any input variable a model trains on, and feature engineering is the deliberate work of constructing better ones from raw data rather than feeding the raw data straight in.
  - Transformations reshape a variable's distribution to better suit a model's assumptions, while interaction features capture that two variables together carry information neither carries alone.
  - A date or timestamp is rarely useful to a model as a raw value; decomposing it into day of week, hour, or time since a reference event usually carries far more signal.
check_questions:
  - q: Why might two variables in combination carry information that neither carries individually?
    a: A model that only ever sees two variables separately cannot represent a rule that depends on their combination — such as a feature that only matters when a price is high and a quantity is also high. An explicit interaction feature makes that combination visible to the model directly.
  - q: Why is a raw timestamp usually a poor feature on its own?
    a: Its raw numeric value — seconds since some epoch — carries almost no usable pattern for most models. Decomposed into day of week, hour of day, or time elapsed since a relevant reference point, the same timestamp usually becomes far more informative.
deeper:
  - label: "IBM Technology — Feature Engineering for AI"
    url: https://www.youtube.com/watch?v=Bg3CjiJ67Cc
  - label: "CampusX — Handling Date and Time Variables"
    url: https://www.youtube.com/watch?v=J73mvgG9fFs
related_formulas: []
---

## Features are constructed, not just collected

A feature is simply an input variable a model sees during training. The data
a project starts with rarely arrives already in the shape a model learns best
from, and feature engineering is the deliberate work of reshaping, combining
and deriving new variables so that the pattern a model needs to find is
easier for it to actually find. This is frequently where the largest
improvement in a project comes from — often larger than switching to a more
sophisticated model, because no model can learn a pattern from information
that was never made available to it in a usable form.

## Transformations: reshaping a distribution

Many models assume or perform better when a variable's distribution is
roughly symmetric rather than heavily skewed. A variable like income or
population, which typically has most values clustered low with a long tail
of unusually large ones, is a common case for a log transformation —
replacing each value with its logarithm compresses the long tail and
produces a far more symmetric, usable distribution. This single technique
appears constantly across real projects precisely because skewed
distributions are themselves extremely common in real-world data.

## Interactions: when two variables matter together

Some patterns only exist in combination. A feature meaning "high price and
high quantity ordered together" is not recoverable by a model that only ever
sees price and quantity as two separate numbers — unless the model is
specifically capable of discovering that interaction on its own, which is not
true of every model type. Explicitly constructing an interaction feature,
such as the product of two existing variables, makes that combined effect
directly visible, rather than leaving it to chance whether the model finds it
unaided.

## Domain-specific features and aggregations

The most valuable features are often the ones that require understanding the
actual problem rather than a generic statistical technique — a ratio that a
domain expert knows matters, a count of how many times a customer has
contacted support in the last month, a rolling average of a sensor's recent
readings rather than its single latest value. Aggregations like these
compress a history of related records into one number a model can use
directly, and they routinely carry more predictive signal than any raw
individual value the aggregation was built from.

## Date and time: decompose before you use it

A raw timestamp is close to useless to most models as a single number — the
numeric value of "seconds since some reference date" carries almost nothing
a model can learn from directly. Decomposed, the same timestamp becomes rich:
day of the week often matters for anything involving human behaviour, hour of
day matters for anything with a daily rhythm, and time elapsed since a
relevant reference event — since a customer's last purchase, say — often
matters more than the absolute date itself. This decomposition step is simple
to implement and is one of the most reliably valuable feature-engineering
moves across an enormous range of problems.

## Where this overlaps with the next lesson

One-hot encoding a categorical variable is sometimes described as a feature-
engineering technique and sometimes as a separate preprocessing step — the
line is somewhat arbitrary, and different teams draw it differently. The
practical dividing line used in this roadmap: constructing genuinely new
information — a ratio, an interaction, a decomposed date — is feature
engineering, and reformatting existing information into a shape a particular
model requires, such as turning a category into numbers, is covered next in
[feature scaling and encoding](/roadmaps/machine-learning/feature-scaling-and-encoding).
