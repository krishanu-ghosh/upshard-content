---
title: Dimensionality reduction
description: Reducing the number of features while preserving structure — PCA, finding the directions of maximum variance via eigenvectors, and what is lost in the process.
roadmap: machine-learning
node: dimensionality-reduction
group: unsupervised-learning
video:
  provider: youtube
  id: "FgakZw6K1QQ"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 1317
level: intermediate
prerequisites: [feature-engineering]
takeaways:
  - PCA finds the directions in the data along which it varies the most, and re-expresses the data in terms of those directions rather than its original features.
  - Keeping only the first few of these directions — the ones capturing the most variance — reduces the number of features substantially while preserving most of what actually varies in the data.
  - The new directions are mathematically guaranteed to be uncorrelated with each other, even when the original features were heavily correlated, which is itself sometimes a useful property independent of the dimensionality reduction.
check_questions:
  - q: What does it mean for PCA to find "the direction of maximum variance"?
    a: Among every possible direction you could draw through the data, it is the one along which the data points are most spread out. Projecting the data onto that single direction preserves more of the original variation than projecting onto any other possible direction.
  - q: What is actually lost when you keep only the first two or three principal components?
    a: Whatever variance existed along the directions that were discarded. If the data's structure genuinely requires more dimensions to represent well, keeping only the top few components discards real information, not just noise — which is why checking how much total variance the kept components explain matters before trusting a reduced representation.
deeper:
  - label: "StatQuest — PCA main ideas in only 5 minutes"
    url: https://www.youtube.com/watch?v=HMOI_lkzW08
  - label: "Luis Serrano Academy — Principal Component Analysis"
    url: https://www.youtube.com/watch?v=g-Hb26agBFg
related_formulas: [singular-value-decomposition, eigenvalue-equation]
---

## Too many features, not enough signal

A dataset with hundreds of features is harder to visualize, slower to train
on, and often contains considerable redundancy — several features that are
really measuring overlapping aspects of the same underlying thing.
Dimensionality reduction addresses this directly: re-express the data using
far fewer numbers while preserving as much of its real structure as possible.
**Principal component analysis (PCA)** is the standard technique for this,
and it is built on a genuinely elegant piece of linear algebra.

## Finding the directions that matter most

PCA's central idea: among every possible direction you could draw through a
dataset, find the one along which the data is most spread out — the
direction of maximum variance. That direction becomes the first **principal
component**. The second principal component is the direction of next-most
variance, with the added constraint that it must be perpendicular to the
first. This continues, each new component capturing as much of the remaining
variance as possible while staying perpendicular to every component found so
far, until there are as many components as original features.

Mechanically, these directions are the **eigenvectors** of the data's
covariance matrix, and the amount of variance each one captures is its
corresponding **eigenvalue** — see
[the eigenvalue equation](/formulas/eigenvalue-equation) and
[singular value decomposition](/formulas/singular-value-decomposition) for
the underlying mathematics that actually computes them.

## Keeping only what matters

Because the components are ordered by how much variance they capture, keeping
only the first few — often just two or three, for visualization, or whatever
number captures some chosen threshold of total variance for other uses —
reduces the dataset to a fraction of its original dimensionality while
retaining most of what actually varied in it. A dataset with two hundred
original features might be well represented by its first ten principal
components if the original features were heavily correlated and genuinely
carried overlapping information; it might need far more components if the
original features were each contributing something largely independent.

## What is actually lost

Discarding the later, lower-variance components discards real information,
not merely noise — if two genuinely distinct clusters in the original data
happened to differ mainly along a direction PCA ranked low, that structure
can be flattened away in a reduced representation that keeps only the top
few components. Checking how much of the total variance the kept components
explain, and not simply assuming two or three components are always enough
for visualization, is the discipline that prevents this from silently
discarding something that mattered.

## Where this is actually used

Beyond visualization, dimensionality reduction speeds up training on
high-dimensional data by giving a model far fewer features to work with,
reduces noise by discarding the lowest-variance directions where noise tends
to concentrate, and can even help detect unusual points, since a genuine
outlier often fails to fit well within the directions that explain most of
the rest of the data — a connection worth keeping in mind heading into the
[anomaly detection](/roadmaps/machine-learning/anomaly-detection) lesson
immediately following this one.
