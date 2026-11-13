---
title: Model selection
description: Comparing different models fairly using the same validation methodology, and the honest case that a simpler, more interpretable model is often the right choice.
roadmap: machine-learning
node: model-selection
group: evaluating-models
video:
  provider: youtube
  id: "SF0YdBXjr_A"
  start: 0
  end: null
creator:
  name: Siddhardhan
  channel_url: https://www.youtube.com/@Siddhardhan
duration_seconds: 863
level: intermediate
prerequisites: [hyperparameter-tuning]
takeaways:
  - Comparing models fairly means evaluating every candidate with the exact same methodology — the same cross-validation folds, the same metric — so differences in the resulting scores reflect the models themselves rather than how each one happened to be evaluated.
  - The model with the single highest score on a validation metric is not automatically the right choice once interpretability, training cost, prediction latency, and maintenance burden are weighed alongside raw accuracy.
  - Choosing a model is the point where every earlier lesson in this roadmap — cleaning, features, the models themselves, honest evaluation — comes together into one final, defensible decision.
check_questions:
  - q: Why does comparing models require using identical cross-validation folds for each one?
    a: If different models were evaluated on different folds, a difference in their scores could be caused by one model simply getting an easier split of the data rather than by any genuine difference in quality. Using the same folds for every model isolates the comparison to the models themselves.
  - q: What is a concrete reason a team might choose a model with a slightly lower validation score over the one that scored highest?
    a: A slightly less accurate but fully interpretable model might be required where a stakeholder or a regulation demands an explanation for every prediction, which a more accurate but opaque model cannot provide no matter how well it scores.
deeper:
  - label: "AMILE / Christian Nabert — Cross Validation for Model Selection"
    url: https://www.youtube.com/watch?v=pCLgW-ex26A
related_formulas: []
---

## The question every earlier lesson has been building toward

This roadmap has covered linear and logistic regression, regularization,
k-nearest neighbors, decision trees, support vector machines, random forests,
and gradient boosting — a real range of models, each with its own strengths.
Model selection is the stage where a concrete problem finally has to choose
among them, using the evaluation tools built up across the previous group to
make that choice on evidence rather than on habit or familiarity with one
particular model.

## Comparing fairly means comparing identically

The single most important discipline in model selection is evaluating every
candidate the same way. If one model is checked with five-fold cross-
validation and another with a single train/test split, any difference in
their reported scores might reflect nothing more than that difference in
methodology rather than any genuine difference in quality. The same folds,
the same metric, and the same preprocessing pipeline applied identically to
every candidate is what makes a comparison between them actually mean
something.

## The highest score is not automatically the right choice

Once every candidate has been evaluated fairly, the model with the single
highest validation score is often, but not always, the right one to ship.
Several other considerations frequently outweigh a small difference in raw
score. **Interpretability** matters enormously in regulated domains, or
anywhere a stakeholder needs a defensible explanation for an individual
prediction — a linear model's coefficients or a decision tree's split logic
can be explained directly, while a large random forest or gradient-boosted
ensemble effectively cannot be, however accurate it is. **Training and
prediction cost** matter in production — a model that must serve predictions
in milliseconds has real constraints a slow but marginally more accurate
model cannot meet. **Maintenance burden** matters over time — a simpler model
is usually easier for a team to understand, debug, and retrain confidently
months after it was first built, long after whoever tuned it has moved on to
other work.

## Choosing simpler on purpose

A genuinely useful habit: when two models perform within a small, practically
insignificant margin of each other on a fair comparison, prefer the simpler
one. The small amount of raw accuracy given up is frequently worth far more
back in interpretability, in lower training and maintenance cost, and in how
much easier the simpler model is to reason about when something eventually
goes wrong in production — and something eventually always does. This is not
a consolation for failing to build the most sophisticated model possible; it
is itself a mature, deliberate engineering decision, and recognising when it
is the right one is a meaningful part of what this whole group of lessons has
been building toward.

## Where this roadmap goes from here

Everything through this point has covered classical models — fit from data
using transparent, explicit mathematical mechanisms a person can trace by
hand. The next group, starting with
[neural network basics](/roadmaps/machine-learning/neural-network-basics),
introduces a different kind of model entirely — one whose power comes
specifically from not committing in advance to a small, interpretable
mathematical form. Everything from this group — honest evaluation, held-out
validation, the risk of overfitting to a validation process — applies just as
directly there, on models that make it considerably harder to see why a
particular prediction came out the way it did.
