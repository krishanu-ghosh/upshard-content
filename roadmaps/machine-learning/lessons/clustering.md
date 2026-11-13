---
title: Clustering
description: Grouping unlabeled data by similarity — k-means and the elbow method, hierarchical clustering and dendrograms, and what "good" means with no labels.
roadmap: machine-learning
node: clustering
group: unsupervised-learning
video:
  provider: youtube
  id: "4b5d3muPQmA"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 510
level: intermediate
prerequisites: [types-of-learning]
takeaways:
  - K-means clustering assigns points to the nearest of k centroids, then moves each centroid to the average of its assigned points, repeating until the assignments stop changing.
  - Hierarchical clustering instead builds a tree of nested groupings from the bottom up, repeatedly merging the two closest clusters, which produces a dendrogram rather than requiring k to be chosen in advance.
  - With no labels to check against, "good clustering" is judged by how tight and well-separated the resulting groups are, not by comparing predictions to ground truth the way supervised learning is evaluated.
check_questions:
  - q: How does k-means decide where to place its centroids?
    a: It starts with an initial guess, assigns every point to its nearest centroid, then recomputes each centroid as the average position of the points assigned to it. This assign-then-recompute cycle repeats until the assignments stop changing between iterations.
  - q: Why can't a clustering result be checked against a "correct answer" the way a classifier's predictions can?
    a: There are no labels in the data to compare against — the whole point of clustering is to find groups nobody has already identified. Quality is instead judged by internal properties — whether points within a cluster are close together and different clusters are well separated from each other.
deeper:
  - label: "StatQuest — Hierarchical Clustering"
    url: https://www.youtube.com/watch?v=7xHsRkOdVwo
related_formulas: [euclidean-norm]
---

## Finding groups nobody labelled

Clustering is the task of grouping data points by similarity with no label
telling the algorithm which group anything actually belongs to — the
unsupervised counterpart to the classification lessons earlier in this
roadmap. Customer segments nobody predefined, or genuinely distinct types of
behaviour hiding inside a single undifferentiated dataset, are the kind of
structure clustering is built to surface.

## K-means: assign, then recompute, repeat

K-means starts by choosing a number of clusters, k, and placing that many
initial centroids — points representing the centre of each cluster — usually
at random. Every data point is then assigned to whichever centroid is
closest to it, using ordinary distance (see
[Euclidean norm](/formulas/euclidean-norm) for the underlying formula). Once
every point has an assignment, each centroid moves to the average position of
all the points now assigned to it. These two steps, assign and recompute,
repeat until the assignments stop changing between iterations, at which point
the algorithm has converged on a final set of clusters.

**Choosing k** is not something the algorithm does on its own — it is a
decision made before the algorithm runs at all. The **elbow method** is the
standard approach: run k-means across a range of values for k, and plot how
much the total within-cluster distance improves as k increases. That
improvement is steep at first and then flattens out, and the point where it
bends — the "elbow" — is a reasonable choice, since adding more clusters
beyond that point buys only a diminishing improvement.

## Hierarchical clustering: no k required up front

Rather than choosing a fixed number of clusters in advance, hierarchical
clustering builds a complete tree of nested groupings. The agglomerative
version, the more common approach, starts with every point as its own
cluster and repeatedly merges the two closest clusters into one, continuing
until everything has merged into a single cluster containing the whole
dataset. The result is a **dendrogram** — a tree diagram showing the full
merge history — which can be cut at any height to produce however many
clusters are wanted after the fact, rather than requiring that choice before
the algorithm ever runs. This flexibility is the main practical advantage
hierarchical clustering has over k-means, at the cost of being considerably
more expensive to compute on a large dataset.

## Judging a clustering with no ground truth

Supervised learning is checked against labels nobody disputes. Clustering has
no such anchor — there is no file somewhere recording the "correct" grouping
to compare against, because if there were, the problem would already be
solved. Quality instead has to be judged by internal properties of the
result itself: are points within a cluster genuinely close together, and are
different clusters well separated from one another? Metrics exist to
quantify exactly this, but they are checking internal consistency, not
agreement with a ground truth, which is a meaningfully different and in some
ways less certain kind of evaluation than anything covered in the
[evaluating models](/roadmaps/machine-learning/evaluation-metrics) group
later in this roadmap.
