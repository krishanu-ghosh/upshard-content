---
title: Decision trees
description: Splitting data recursively on feature thresholds to build a tree — how splits are chosen, overfitting via depth, and why a tree is interpretable but unstable.
roadmap: machine-learning
node: decision-trees
group: supervised-models
video:
  provider: youtube
  id: "_L39rN6gz7Y"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 1088
level: beginner
prerequisites: [logistic-regression]
takeaways:
  - A decision tree splits the data recursively, at each step choosing the feature and threshold that produce the most informative separation, until a stopping condition is reached.
  - Gini impurity measures how mixed the classes are at a node, and the split chosen at each step is the one that reduces impurity the most across the resulting branches.
  - A tree grown deep enough fits its training data almost perfectly and badly overfits in the process; limiting depth is the most direct way to control this.
check_questions:
  - q: What does a split at a decision tree node actually choose?
    a: A single feature and a threshold on that feature — for example, "is age greater than 40" — chosen because it separates the data at that node into two groups that are each more homogeneous in their outcome than the group was before the split.
  - q: Why is a single decision tree described as unstable?
    a: A small change in the training data — removing or adding a handful of examples — can change which split looks best near the top of the tree, which cascades into a very different tree structure below it, even though the underlying pattern in the data barely changed at all.
deeper:
  - label: "Normalized Nerd — Decision Tree Classification Clearly Explained"
    url: https://www.youtube.com/watch?v=ZVR2Way4nwQ
related_formulas: []
---

## Learning by asking questions

A decision tree predicts by asking a sequence of yes-or-no questions about
the input, with each answer leading to a different branch, until a final
answer is reached at a leaf. "Is income above fifty thousand? If yes, is age
above forty? If yes, predict approved." This is close to how a person might
actually reason through a decision by hand, which is a large part of why
trees are considered interpretable in a way few other models are — the
entire chain of reasoning behind any single prediction can be read off
directly, split by split.

## How a split is actually chosen

At each node, the tree considers every feature and every possible threshold
on that feature, and picks whichever single split makes the data on each side
most homogeneous with respect to the outcome being predicted. **Gini
impurity** is the standard way to measure that homogeneity for classification:
a node where every example belongs to the same class has a Gini impurity of
zero, and a node with an even mix of classes has a high one. The split chosen
at each step is whichever one reduces this impurity the most across the two
resulting branches, and the tree repeats this process recursively on each
branch, splitting again and again until it reaches a stopping condition.

## Depth is the main overfitting control

A tree allowed to grow without limit will keep splitting until every leaf
contains a single training example, perfectly memorising the training data in
the process — and badly overfitting, since real-world new data will not match
that exact memorised structure. Limiting the tree's maximum depth, or
requiring a minimum number of examples before a further split is allowed, are
the standard ways to prevent this: both cap how finely the tree is permitted
to carve up the training data, trading some training accuracy for a model
that generalises better to new data, exactly the bias-variance tradeoff
covered in
[generalization and overfitting](/roadmaps/machine-learning/generalization-and-overfitting).

## Interpretable, and unstable

A tree's full decision logic can be read directly, split by split, which
makes it one of the most interpretable models in this roadmap — a real
advantage whenever a stakeholder needs to understand not just what a model
predicted but why. The cost is **instability**: because each split is chosen
greedily based on what looks best at that one node, a small change in the
training data — adding or removing a handful of examples — can change which
split looks best near the top of the tree, and that change cascades into a
substantially different tree below it, even when the underlying pattern in
the data has barely shifted at all.

This specific weakness is exactly what motivates combining many trees
together rather than relying on one, which is the whole idea behind
[bagging and random forests](/roadmaps/machine-learning/bagging-and-random-forests) in the next group: average over many unstable trees, each
trained on a slightly different view of the data, and the instability of any
individual tree washes out in the average.
