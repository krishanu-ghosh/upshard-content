---
title: Anomaly detection
description: Finding rare, unusual data points with statistical, density and distance-based approaches — and the challenge of validating with few or no labeled anomalies.
roadmap: machine-learning
node: anomaly-detection
group: unsupervised-learning
video:
  provider: youtube
  id: "XzEXB12N1xs"
  start: 0
  end: null
creator:
  name: Krish Naik
  channel_url: https://www.youtube.com/@krishnaik06
duration_seconds: 984
level: intermediate
prerequisites: [clustering, dimensionality-reduction]
takeaways:
  - Statistical methods flag a point as anomalous based on how far it sits from the rest of the data's distribution — a z-score or the interquartile range are the two most common measures.
  - Density- and distance-based methods, such as isolation forests and local outlier factor, instead look at how isolated a point is from its neighbours, which catches anomalies that a simple statistical threshold on one variable alone would miss.
  - Anomalies are by definition rare, which means there is usually little or no labeled data to validate a detector against, unlike almost every other model covered in this roadmap.
check_questions:
  - q: What is the limitation of a z-score or IQR-based approach to finding outliers?
    a: These methods look at one variable at a time, so they miss an anomaly that looks perfectly normal on every individual variable but is unusual in combination — a value that is ordinary for its age group and ordinary for its income bracket, but highly unusual for that specific combination of the two.
  - q: Why is validating an anomaly detector harder than validating a typical classifier?
    a: A typical classifier is checked against a labeled test set with a reasonable number of examples of every class. Anomalies are rare by definition, so there is often little or no labeled data confirming which points are genuinely anomalous, leaving far less to validate a detector's performance against.
deeper:
  - label: "AiML Mastery Club — Isolation Forest, Clearly Explained"
    url: https://www.youtube.com/watch?v=kqAxfOPlr1U
  - label: "Aric LaBarr — What is the Local Outlier Factor?"
    url: https://www.youtube.com/watch?v=Ymvq6JHjoBY
related_formulas: [variance, normal-distribution]
---

## Finding the point that does not belong

Anomaly detection looks for data points that differ meaningfully from the
rest of the dataset — a fraudulent transaction among millions of normal ones,
a failing sensor reading among millions of healthy ones. Unlike every
classification model covered earlier in this roadmap, there is often little
or no labeled data confirming which specific historical points were genuine
anomalies, which shapes both how these methods work and how they get
evaluated.

## Statistical approaches: how far from typical

The simplest methods look at how far a value sits from the rest of its own
distribution. A **z-score** measures a value in terms of how many standard
deviations it sits from the mean — a value with a z-score of 4 is extremely
far into the tail of a roughly normal distribution and a natural candidate
for flagging. The **interquartile range (IQR)** approach instead measures the
spread between the 25th and 75th percentiles of the data and flags anything
falling well outside that central range, which handles skewed distributions
somewhat more gracefully than a z-score built on the mean and standard
deviation, since it does not assume the data is symmetric.

## Where a single-variable threshold misses the point

Both statistical methods above look at one variable at a time, which misses
an entire category of anomaly: a point that looks completely ordinary on
every individual variable but is highly unusual in combination. A
transaction amount that is normal, from an account that is normal, at a time
of day that is normal, can still be a clear anomaly if that specific
combination of amount, account, and timing has never occurred together
before. This is exactly what density- and distance-based methods are built to
catch.

## Density and distance: how isolated is this point

An **isolation forest** detects anomalies by a clever inversion of ordinary
tree-building: it randomly selects a feature and a split point, repeatedly,
and measures how many splits it takes to isolate a given point from the rest
of the data. A normal point, surrounded by many similar neighbours, typically
takes many splits to isolate. An anomalous point, sitting apart from the
bulk of the data, tends to get isolated in very few splits — the ease of
isolation is itself the anomaly score.

The **local outlier factor (LOF)** instead compares a point's local density —
how packed together its nearby neighbours are — to the local density around
those same neighbours. A point sitting in a notably sparser neighbourhood
than its neighbours' own neighbourhoods is flagged as an outlier, even if it
is not far in absolute distance from the rest of the dataset, which makes LOF
particularly good at finding anomalies in data where normal density varies
across different regions.

## The validation problem this lesson flags and does not solve

Every model covered earlier in this roadmap is validated by checking
predictions against labeled outcomes nobody disputes. Anomaly detection
frequently cannot do this at all: if an organisation already had reliable
labels for every past anomaly, much of the detection problem would already be
solved. In practice, an anomaly detector is often deployed and monitored
based on investigation of what it actually flags after the fact, rather than
measured cleanly against a held-out labeled test set the way a classifier is
in the [evaluating models](/roadmaps/machine-learning/evaluation-metrics)
group that follows — a genuine limitation of this entire category of
technique, not a gap specific to any one method covered here.
