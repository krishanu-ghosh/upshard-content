---
title: Support vector machines
description: Finding the maximum-margin hyperplane separating classes, the kernel trick for non-linear data, and the C hyperparameter controlling the tradeoff.
roadmap: machine-learning
node: support-vector-machines
group: ensembles-and-kernels
video:
  provider: youtube
  id: "Lpr__X8zuE8"
  start: 0
  end: null
creator:
  name: Luis Serrano Academy
  channel_url: https://www.youtube.com/@SerranoAcademy
duration_seconds: 1857
level: intermediate
prerequisites: [k-nearest-neighbors, regularization]
takeaways:
  - A support vector machine finds the dividing line (or hyperplane) between two classes that maximizes the margin — the distance to the nearest point of either class — rather than just any line that separates them.
  - Only the points closest to that boundary, the support vectors, actually determine where it sits; every other point could move or vanish without changing the result at all.
  - The kernel trick lets a support vector machine separate data that is not linearly separable in its original space, by implicitly working in a higher-dimensional space where a straight boundary does exist.
check_questions:
  - q: Why does a support vector machine prefer the boundary with the maximum margin rather than any boundary that separates the classes?
    a: A boundary with a wide margin generalizes better to new points near the edge of either class, because it has the most room for error before misclassifying something close to the training examples. A boundary that barely separates the training data is fragile to small shifts in new data.
  - q: What does the kernel trick actually let the model do that a plain linear boundary cannot?
    a: Some datasets have no straight line or flat hyperplane that separates the classes in their original features. A kernel implicitly maps the data into a higher-dimensional space where such a straight separation does exist, without ever explicitly computing the coordinates in that higher-dimensional space.
deeper:
  - label: "StatQuest — Support Vector Machines Part 1: Main Ideas"
    url: https://www.youtube.com/watch?v=efR1C6CvhmE
  - label: "StatQuest — Support Vector Machines Part 2: The Polynomial Kernel"
    url: https://www.youtube.com/watch?v=Toet3EiSFcM
related_formulas: [dot-product]
---

## Not just any separating line

Given two classes that can be separated by a straight line, there are
infinitely many lines that would do the job. A support vector machine does
not settle for any of them — it finds the one that maximizes the **margin**,
the distance from the boundary to the closest point of either class. The
intuition for why this specific choice matters: a boundary squeezed tight
against the training data has almost no room for error on a new point that
falls slightly differently than the training examples did, while a boundary
with the widest possible margin has the most buffer before a new point near
the edge gets misclassified.

## Support vectors: only the closest points matter

The points that sit exactly on the margin's edge are called **support
vectors**, and they are the only points that actually determine where the
boundary sits. Every other point in the training set — anything further from
the boundary than the support vectors — could be moved, or removed entirely,
without changing the fitted boundary at all. This is a genuinely distinctive
property among the models in this roadmap: most models are influenced by
every training point to some degree, while a support vector machine's
decision boundary depends on only a small subset of them. In practice this
means a dataset with a thousand points and only a dozen support vectors
produces exactly the same boundary as one where the other 988 points were
never collected at all — the boundary genuinely does not depend on them.

## The C parameter: how strict is the margin

Real data is rarely perfectly separable, and forcing a hard boundary with
zero misclassifications on noisy data usually produces a narrow, fragile
margin chasing a handful of difficult points. The **C** parameter controls
this tradeoff directly: a large C penalizes any misclassified training point
heavily, producing a narrower margin that fits the training data closely; a
small C tolerates more misclassified points in exchange for a wider margin
that holds up better on new data. This is conceptually the same bias-variance tradeoff that runs
through [regularization](/roadmaps/machine-learning/regularization) — C is,
in effect, a support vector machine's own regularization strength, and it
needs to be chosen the same way, by checking performance on held-out data
rather than guessed.

## The kernel trick: separating what a straight line cannot

Some datasets have no straight boundary that separates their classes at all
— imagine one class forming a ring around the other. The **kernel trick**
handles this by implicitly mapping the data into a higher-dimensional space
where a flat boundary does exist, without ever explicitly computing the
coordinates of that higher-dimensional space, which would often be
prohibitively expensive. A **polynomial kernel** and a **radial basis
function (RBF) kernel** are the two most common choices, each defining a
different notion of similarity between points under the hood — mechanically,
a kernel is a function that computes what the [dot product](/formulas/dot-product) between two points would be in that higher-dimensional space,
directly from their original coordinates, which is the mathematical sleight
of hand that makes the whole trick computationally feasible.
