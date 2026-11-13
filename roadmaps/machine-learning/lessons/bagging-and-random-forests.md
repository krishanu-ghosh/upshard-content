---
title: Bagging and random forests
description: Training many models on resampled data and aggregating their predictions — and why random forests also randomize which features each tree can split on.
roadmap: machine-learning
node: bagging-and-random-forests
group: ensembles-and-kernels
video:
  provider: youtube
  id: "v6VJ2RO66Ag"
  start: 0
  end: null
creator:
  name: Normalized Nerd
  channel_url: https://www.youtube.com/@NormalizedNerd
duration_seconds: 481
level: intermediate
prerequisites: [decision-trees]
takeaways:
  - Bagging (bootstrap aggregating) trains many models, each on a different random resample of the training data, and averages or votes across their predictions to reduce variance.
  - A random forest is bagging applied specifically to decision trees, with one addition — at every split, only a random subset of features is even considered, not all of them.
  - That extra randomization decorrelates the trees from each other, which matters because averaging reduces variance far more effectively across predictions that are not all making the same mistakes.
check_questions:
  - q: Why does averaging predictions from many models reduce variance, even if each individual model is still prone to overfitting?
    a: Each model overfits to the particular noise in its own resampled training set, and that noise differs from model to model. Averaging cancels out noise that disagrees across models while preserving the genuine signal that every model, trained on overlapping real data, tends to agree on.
  - q: Why does a random forest restrict each split to a random subset of features, rather than letting every tree consider all of them?
    a: Without this restriction, a strong single feature would dominate the top split of nearly every tree, making the trees highly similar to each other and correlated. Averaging over correlated trees barely reduces variance; restricting the features considered at each split forces the trees to differ more from one another, which is what makes the averaging actually work.
deeper:
  - label: "StatQuest — Random Forests Part 1: Building, Using and Evaluating"
    url: https://www.youtube.com/watch?v=J4Wdy0Wc_xQ
  - label: "Krish Naik — What is Bagging (Bootstrap Aggregation)?"
    url: https://www.youtube.com/watch?v=KIOeZ5cFZ50
related_formulas: []
---

## The idea behind bagging

A single decision tree, as the previous lesson covered, is unstable — a small
change to the training data can produce a noticeably different tree.
**Bagging**, short for bootstrap aggregating, turns this instability into an
advantage rather than fighting it directly: train many trees, each on its own
random resample of the training data (drawn with replacement, so some
examples appear more than once in a given resample and others are left out
entirely), and combine their predictions by averaging for a continuous
output or by majority vote for a classification. Each individual tree still
overfits to the particular noise in its own resample, but that noise differs
from tree to tree, while the real underlying signal, present throughout the
original data, tends to agree across every resample. Averaging cancels the
former while preserving the latter.

Drawing with replacement has a useful side effect: each resample leaves out
roughly a third of the original examples on average, purely by chance. Those
left-out examples, called **out-of-bag** examples, were never seen by the
tree trained on that particular resample, so scoring each tree only on the
examples it did not train on gives a free, built-in estimate of how well the
ensemble generalises — without setting aside a separate validation set at all.

## Random forests: bagging plus one more trick

A **random forest** is bagging applied specifically to decision trees, with
one further addition. At every single split, rather than letting a tree
consider every available feature, it is restricted to a random subset of
them. This might seem like it should hurt each individual tree, and in isolation
it does. But it serves a more important purpose across the ensemble as a
whole: without this restriction, any one strong, obviously useful feature
would dominate the top split of nearly every tree in the forest, making the
trees highly similar to each other. Averaging over similar, correlated trees
barely reduces variance at all, because they are all making roughly the same
mistakes in roughly the same places. Forcing each tree to work with a
different random subset of features at each split makes the trees genuinely
differ from one another — precisely the condition under which averaging
delivers a large reduction in variance.

## Why decorrelation is the whole point

This is worth sitting with directly, because it is easy to average over many
models and assume the benefit comes simply from "more models." It does not —
averaging over many highly correlated models, all making essentially the same
errors, reduces variance only slightly. Averaging over many decorrelated
models, each making different errors that are not all pointing the same
direction, reduces variance substantially, because the errors genuinely
cancel rather than reinforcing each other. This is the single most important
idea in this lesson, and it is the reason random forests reliably outperform
plain bagged trees by a meaningful margin in practice, despite the two
methods looking almost identical on paper.

## What a forest trades away

A random forest trades the single tree's transparent, readable decision logic
for better, more stable predictions — a forest of a few hundred trees has no
simple chain of questions a person can read off the way a single tree does.
This tradeoff, accuracy and stability against interpretability, recurs
throughout this roadmap, and it is also exactly the trade that
[gradient boosting](/roadmaps/machine-learning/gradient-boosting), the final
lesson in this group, makes in a different way.
