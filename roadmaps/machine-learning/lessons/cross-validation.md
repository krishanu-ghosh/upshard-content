---
title: Cross-validation
description: Why a single train/test split can mislead, k-fold and stratified k-fold mechanics, and leave-one-out as the extreme case.
roadmap: machine-learning
node: cross-validation
group: evaluating-models
video:
  provider: youtube
  id: "PF2wLKv2lsI"
  start: 0
  end: null
creator:
  name: Mahesh Huddar
  channel_url: https://www.youtube.com/@MaheshHuddar
duration_seconds: 540
level: intermediate
prerequisites: [evaluation-metrics, gradient-boosting]
takeaways:
  - A single train/test split gives one estimate of performance that depends on which particular examples happened to land in the test set — an unlucky split can make a good model look mediocre or a mediocre model look good.
  - K-fold cross-validation splits the data into k parts, trains and tests k times with a different part held out each time, and averages the results into a far more reliable estimate.
  - Stratified k-fold preserves the proportion of each class in every fold, which matters a great deal on imbalanced data where a plain random fold could easily end up with too few examples of the rare class.
check_questions:
  - q: Why does a single train/test split give an unreliable estimate of a model's true performance?
    a: The specific examples that happen to land in the test set affect the measured performance, and a different random split of the same data could easily produce a noticeably different score — the estimate depends partly on luck, not purely on the model's actual quality.
  - q: Why does stratified k-fold matter more on an imbalanced dataset than on a balanced one?
    a: A plain random fold could easily, by chance, end up with very few or even zero examples of a rare class, making that fold's evaluation nearly meaningless for that class. Stratified k-fold deliberately preserves the overall class proportions in every fold, avoiding this.
deeper:
  - label: "StatQuest — Cross Validation"
    url: https://www.youtube.com/watch?v=fSytzGwwBVw
related_formulas: []
---

## One split is one roll of the dice

A single train/test split, covered in
[generalization and overfitting](/roadmaps/machine-learning/generalization-and-overfitting), produces exactly one estimate of how well a model performs —
and that estimate depends partly on which specific examples happened to land
in the test set. An unusually easy test set makes a mediocre model look good;
an unusually hard one makes a good model look mediocre. Neither outcome tells
you reliably what will happen on the next batch of genuinely new data,
because the measurement itself carries real variance from the luck of the
split.

## K-fold: averaging over many splits instead of trusting one

**K-fold cross-validation** fixes this by not relying on a single split at
all. The data is divided into k roughly equal parts, called folds. The model
is trained k separate times, each time on k-1 of the folds combined, and
tested on whichever single fold was held out that round. Every fold gets used
as the test set exactly once across the k rounds, and the final performance
estimate is the average across all k results. This single number is far more
reliable than any one split's result, because it is no longer sensitive to
which particular examples happened to land in any one test set — every
example contributes to the test score exactly once across the full process.

## Stratified k-fold: keeping class balance honest

Plain k-fold splits examples into folds without regard to class balance,
which is a real problem on imbalanced data: a fold could, purely by chance,
end up with very few or even zero examples of a rare class, making that
fold's test score close to meaningless for judging performance on exactly the
class that matters most. **Stratified k-fold** fixes this by deliberately
preserving the overall proportion of each class in every single fold, so a
dataset that is ten percent positive overall produces folds that are each
also close to ten percent positive, rather than leaving that to chance. This
connects directly back to the
[imbalanced data](/roadmaps/machine-learning/imbalanced-data) lesson — any
project dealing with class imbalance should default to stratified k-fold
rather than plain k-fold, essentially without exception.

## Leave-one-out and leave-p-out: the extreme case

**Leave-one-out cross-validation** is k-fold taken to its limit: k is set
equal to the total number of examples, so every single round trains on all
but one example and tests on that one held-out example alone. This uses the
data as thoroughly as possible, since nearly the entire dataset is always
available for training, but it is extremely expensive computationally on
anything but a small dataset, since it requires training a separate model
once for every single example. **Leave-p-out** generalises this further,
holding out every possible combination of p examples at a time rather than
just one, which grows combinatorially expensive even faster and is rarely
used in practice beyond very small values of p on very small datasets.

## What cross-validation is actually for

It is worth being precise about what this technique estimates and does not.
Cross-validation gives a far more reliable estimate of how well a given
model configuration will generalise, which makes it the right tool for
comparing configurations or tuning settings — exactly the use covered next in
[hyperparameter tuning](/roadmaps/machine-learning/hyperparameter-tuning). It
is not a substitute for a final, untouched test set: once a configuration has
been chosen using cross-validation, reporting its performance still properly
belongs on data that played no role whatsoever in that selection process.
