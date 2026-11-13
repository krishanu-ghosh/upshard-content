---
title: Model evaluation and validation
description: Offline metrics, holdout sets, and the gate a model has to clear between "it trained" and "it's allowed to ship."
roadmap: mlops
node: model-evaluation-and-validation
group: experimentation
video:
  provider: youtube
  id: "LbX4X71-TFI"
  start: 0
  end: null
creator:
  name: AssemblyAI
  channel_url: https://www.youtube.com/@AssemblyAI
duration_seconds: 604
level: intermediate
prerequisites: [model-registries, feature-stores]
takeaways:
  - A model that trains successfully and a model that is good enough to deploy are different claims, and validation is the explicit gate that tests the second claim, not just the first.
  - Evaluation on a holdout or test set estimates how a model performs on data it never saw during training, which is the only honest proxy available for real-world performance before deployment.
  - A single aggregate metric like overall accuracy can hide serious failures on specific slices of data, so validation that only checks one number can pass a model that's badly broken for a subgroup it matters for.
check_questions:
  - q: Why isn't training accuracy a trustworthy signal of how well a model will perform in production?
    a: Because a model can fit its training data well through memorization or overfitting without generalizing to data it hasn't seen, so training accuracy mainly measures fit, not generalization.
  - q: What is a holdout or test set, and why must it stay separate from training and tuning?
    a: It's a portion of labeled data withheld from both training and hyperparameter tuning, kept aside purely to give an unbiased estimate of performance on unseen data; using it during training or tuning lets its information leak into the model and inflates the estimate.
  - q: Why can a model pass on overall accuracy but still fail a validation gate?
    a: Because overall accuracy averages performance across the whole dataset and can hide much worse performance on a specific slice or subgroup that a slice-level check would catch but an aggregate number conceals.
deeper:
  - label: "Machine Learning Fundamentals: Cross Validation — StatQuest with Josh Starmer"
    url: https://www.youtube.com/watch?v=fSytzGwwBVw
related_formulas: []
---

## "It trained" is not a claim about quality

A training run finishing without errors and producing a model file tells
you almost nothing about whether that model is any good. Loss went down,
sure — but down compared to what, and does that translate to useful
real-world performance? Evaluation and validation exist to answer a
specific, narrower question than "did training succeed": is this
particular trained model good enough, by defined criteria, to be allowed
into production? Those are genuinely different questions. A model can
train cleanly and still be useless — overfit to its training data,
blind to an entire category of input it will see in production, or simply
worse than the model it's meant to replace. Validation is the deliberate
step that stands between "a model exists" and "a model ships," and
skipping it means the first time anyone actually checks whether the model
is good is after it's already live.

## Why you need a held-out test set

The core mechanism is simple but easy to get wrong in practice: you can't
evaluate a model on the same data it trained on, because a model has every
opportunity to memorize patterns specific to that data rather than learn
patterns that generalize. A decision tree with enough depth, for instance,
can achieve near-perfect accuracy on its training set by essentially
encoding individual examples — and still perform badly on new data because
it learned noise, not signal. The fix is to hold out a slice of labeled
data — the test set — that the model never sees during training, and
never sees during hyperparameter tuning either (tuning uses a separate
validation split for that purpose, precisely so the test set stays
untouched as the final, honest check). Evaluating only on this held-out
data gives you an estimate of how the model performs on data resembling
what it will actually encounter after deployment, because it's the
closest approximation available to "data the model hasn't memorized."

This is also why cross-validation exists as a refinement: instead of one
fixed train/test split, which can give a lucky or unlucky estimate
depending on which examples happened to land in which split, you rotate
through several splits and average the result, trading more compute for a
more stable estimate of generalization performance.

## Why one number can lie

Suppose a fraud model scores 98% overall accuracy on its test set — a
number that looks excellent until you remember that if only 1% of real
transactions are fraudulent, a model that predicts "not fraud" for
everything scores 99% while catching zero actual fraud. Aggregate
accuracy against an imbalanced dataset is a classic way a single metric
hides the thing you actually care about. The same failure shows up more
subtly across demographic or behavioral slices: a model might score 95%
overall while performing at 70% for a specific customer segment, because
that segment is underrepresented in the training data and the aggregate
metric simply averages its worse performance away. A validation gate that
only checks the headline number lets this kind of model through
unnoticed; a validation gate worth trusting checks performance on the
specific slices that matter for the use case — not just precision and
recall overall, but broken down by the segments where a failure would
actually cost something.

## What a validation gate looks like in practice

Put concretely, a validation step for a newly registered candidate model
(picking up where the model registry lesson left off) typically checks:
performance against the held-out test set, compared against a defined
minimum threshold; performance against the currently deployed production
model, so a new version can't silently regress even if it clears the
absolute threshold; and performance broken down across the slices that
matter for this specific use case. Only a model that clears all three gets
promoted in the registry; one that fails stays in `Staging`, with the
failure recorded so whoever's tuning it next knows exactly what to fix.
This is what turns "good enough" from a judgment call made once in a
notebook into a repeatable, auditable check applied the same way every
time.

## Why this matters going forward

Evaluation and validation is the last checkpoint experimentation produces
before a model is treated as production-ready — it's the gate the rest of
this roadmap's packaging, CI/CD, and deployment lessons assume has already
been cleared. Monitoring, covered much later, picks up exactly where this
leaves off: validation checks a model against a fixed test set before
deployment, while monitoring checks whether the live world still resembles
that test set after deployment. Neither replaces the other — a model that
passed validation can still degrade once real data drifts away from what
the test set captured, which is precisely why this gate is necessary but
never sufficient on its own.
