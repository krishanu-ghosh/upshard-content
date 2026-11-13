---
title: Explainability and interpretability
description: SHAP and LIME — how each technique explains why a model made a specific prediction, for debugging drift and for building trust.
roadmap: mlops
node: explainability-and-interpretability
group: monitoring-governance
video:
  provider: youtube
  id: "9haIOplEIGM"
  start: 0
  end: null
creator:
  name: DeepFindr
  channel_url: https://www.youtube.com/@DeepFindr
duration_seconds: 950
level: advanced
prerequisites: [monitoring-model-performance]
takeaways:
  - SHAP assigns each feature a contribution to one specific prediction by averaging that feature's marginal effect across every possible order features could be "added" to the model, giving attributions that are mathematically guaranteed to sum to the gap between the prediction and the average prediction.
  - LIME takes a different approach to explaining a single prediction — it perturbs the input many times, observes how the model's output changes, and fits a simple, interpretable model to that local neighborhood, trading SHAP's theoretical guarantees for speed and model-agnostic simplicity.
  - Explainability isn't just for compliance paperwork — it's a debugging tool, since watching which features drive a drifted or wrong prediction often reveals the actual root cause faster than staring at aggregate accuracy numbers.
check_questions:
  - q: What does a SHAP value represent for a given feature and a given prediction?
    a: That feature's average marginal contribution to pushing the prediction away from the baseline (average) prediction, computed by averaging its effect across every possible ordering in which features could be considered.
  - q: How does LIME generate its explanation for a single prediction?
    a: It creates many slightly perturbed versions of that one input, runs them through the real model to see how the output changes, and fits a simple interpretable model (like a linear model) to those nearby points, using that local fit as the explanation.
  - q: Why might two correct predictions still need separate SHAP explanations rather than one global explanation?
    a: Because SHAP explains individual predictions, and the features driving one prediction (say, a high-income applicant's approval) can be entirely different from the features driving another correct prediction for a different applicant, even though both were classified correctly.
deeper:
  - label: "Shapley Values : Data Science Concepts — ritvikmath"
    url: "https://www.youtube.com/watch?v=NBg7YirBTN8"
related_formulas: []
---

## The question a confidence score can't answer

A model can output "approved, 91% confidence" and that number tells you
nothing about *why*. Was it the applicant's income? Their credit history?
Some spurious correlation with their zip code that the model picked up
from biased training data? **Explainability** and **interpretability**
are the set of techniques for answering that "why" for a specific
prediction, rather than for the model's behavior in aggregate — and that
distinction matters: a model that's 95% accurate overall can still be
making its correct predictions for subtly wrong reasons on a worrying
subset of cases, and aggregate accuracy will never surface that.

## SHAP: borrowing a 70-year-old fairness idea from game theory

**SHAP** (SHapley Additive exPlanations) is built on *Shapley values*, a
concept from cooperative game theory originally designed to answer a
different question: if a group of players cooperate to produce some total
payoff, how should that payoff be fairly split among them, given that
different players contribute differently depending on who else is already
in the coalition? SHAP reframes a prediction as a "game" where the
features are the players and the "payoff" is the gap between this specific
prediction and the model's average prediction over the whole dataset. For
each feature, SHAP asks: across every possible order in which features
could be revealed to the model one at a time, how much does adding *this*
feature change the prediction, on average? That average marginal
contribution is the feature's SHAP value for this prediction.

The reason this is worth the computational cost (exact Shapley values
require considering every subset of features, which is why practical SHAP
implementations use sampling or model-specific shortcuts like TreeSHAP for
tree-based models) is the guarantees it buys. SHAP values are *additive*:
sum every feature's SHAP value for a prediction, add the baseline average
prediction, and you get back exactly the model's actual output for that
case. No other common attribution method gives you that exact accounting.
It also satisfies *consistency*: if you change a model so a feature
matters more, that feature's SHAP value can only go up, never down — a
property that sounds obvious but that several earlier feature-importance
techniques actually violate.

## LIME: a cheaper, local approximation

**LIME** (Local Interpretable Model-agnostic Explanations) takes a more
pragmatic route to the same goal. Rather than reasoning about every
possible feature ordering, LIME picks the one prediction it wants to
explain, generates a cloud of synthetic neighbors by perturbing that
input's features slightly (flipping words in text, blurring patches in an
image, jittering numeric features in tabular data), runs the real model on
all of them, and then fits a simple, interpretable model — typically a
weighted linear regression — to that local cloud of inputs and outputs.
The coefficients of that local linear model become the explanation: this
feature pushed the prediction up, that one pushed it down, in the
immediate neighborhood of this specific input. LIME makes no claim about
global consistency the way SHAP does — its explanation is honestly only
valid in a small neighborhood around one point — but it's model-agnostic,
conceptually simple, and often considerably cheaper to compute, which is
why it remains a common first tool, especially for unstructured data like
text and images.

## Explainability as a debugging instrument, not just a report

The practical payoff shows up well before any compliance review. When a
drift alert fires or an incident report needs a root cause, computing SHAP
or LIME explanations on the affected predictions is often the fastest way
to see what actually changed — a feature that used to carry little weight
suddenly dominating every explanation is a strong, specific clue about
what's gone wrong in the data, in a way that a single aggregate accuracy
number never could be.

## Where this leads next

Explainability is also the mechanism that makes the next lesson's
governance requirements achievable rather than aspirational: a regulation
that requires a lender to state why an applicant was denied, or an
internal policy that requires a human reviewer to sanity-check automated
decisions, is only satisfiable if the model can actually produce a
faithful per-prediction explanation on demand. SHAP and LIME are the two
techniques that make that demand answerable.
