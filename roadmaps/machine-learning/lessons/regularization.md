---
title: Regularization
description: L1 (Lasso) and L2 (Ridge) regularization — why unconstrained models overfit, how penalizing large coefficients helps, and the difference between the two.
roadmap: machine-learning
node: regularization
group: supervised-models
video:
  provider: youtube
  id: "9hU2RZ_Y3DA"
  start: 0
  end: null
creator:
  name: Gate Smashers
  channel_url: https://www.youtube.com/@GateSmashers
duration_seconds: 850
level: intermediate
prerequisites: [linear-regression, logistic-regression]
takeaways:
  - Regularization adds a penalty for large coefficients to the training objective, trading a little fit on the training data for a model less likely to have overfit it.
  - L2 (Ridge) shrinks every coefficient toward zero smoothly, while L1 (Lasso) can shrink a coefficient all the way to exactly zero, which amounts to automatic feature selection.
  - The strength of the penalty is itself a hyperparameter — too little and overfitting is barely addressed, too much and the model underfits by shrinking genuinely useful coefficients away.
check_questions:
  - q: Why does penalizing large coefficients help with overfitting?
    a: An overfit model often relies on a few large coefficients to chase noise specific to the training set. Penalizing large coefficients discourages the model from leaning that heavily on any single input, producing a smoother fit that generalizes better.
  - q: Why can L1 regularization be used for feature selection but L2 generally cannot?
    a: L1's penalty can push a coefficient all the way to exactly zero, which removes that feature's influence on the model entirely. L2's penalty shrinks coefficients smoothly toward zero without usually reaching it exactly, so every feature keeps some small influence.
deeper:
  - label: "StatQuest — Regularization Part 1: Ridge (L2) Regression"
    url: https://www.youtube.com/watch?v=Q81RR3yKn30
  - label: "StatQuest — Ridge vs Lasso Regression, Visualized"
    url: https://www.youtube.com/watch?v=Xm2C_gTAl8c
related_formulas: [l1-regularization, l2-regularization]
---

## The problem regularization solves

A model with enough flexibility can fit its training data almost perfectly,
including the noise specific to that one dataset — the overfitting problem
covered in
[generalization and overfitting](/roadmaps/machine-learning/generalization-and-overfitting). One common symptom of this is unusually large coefficients:
a model chasing noise often does so by leaning heavily on one or two inputs
with large weights, producing predictions that swing wildly in response to
small changes in those inputs. Regularization directly discourages this, by
adding a penalty for coefficient size into the quantity the model is trained
to minimise, alongside the usual prediction error.

## L2, or Ridge: shrink everything a little

[L2 regularization](/formulas/l2-regularization) adds a penalty proportional
to the sum of the squared coefficients. Minimising prediction error plus this
penalty together pushes every coefficient smaller than it would otherwise be,
smoothly, without driving any of them exactly to zero. This produces a model
that relies a little less heavily on any single input, trading a small amount
of fit on the training data for a model expected to generalise better to new
data.

## L1, or Lasso: shrink some all the way to zero

[L1 regularization](/formulas/l1-regularization) adds a penalty proportional
to the sum of the absolute values of the coefficients, rather than their
squares. The practical consequence is different in kind, not just degree: L1's
penalty can push a coefficient all the way to exactly zero, effectively
removing that feature from the model entirely. Applied to a dataset with many
candidate features, L1 regularization therefore acts as automatic feature
selection — the features that survive with a non-zero coefficient are the
ones the model found genuinely useful, and the rest are cut out entirely
rather than merely shrunk.

## Choosing between them, and choosing the strength

Prefer L1 when a dataset has many candidate features and a genuine suspicion
that only some of them matter — the automatic feature selection is directly
valuable there. Prefer L2 when most features are at least somewhat
informative and the goal is simply a smoother, less overfit model rather than
a sparser one. Some approaches, often called elastic net, combine both
penalties at once rather than forcing a choice between them.

Either way, the strength of the penalty — how heavily large coefficients are
punished — is itself a setting the model does not learn on its own; it has to
be chosen, typically by trying several strengths and checking which performs
best on held-out data, exactly the kind of decision
[cross-validation](/roadmaps/machine-learning/cross-validation) and
[hyperparameter tuning](/roadmaps/machine-learning/hyperparameter-tuning)
are built to make systematically. Too little penalty barely addresses
overfitting at all; too much shrinks away coefficients that were genuinely
useful, trading overfitting for underfitting instead of finding the balance
between them.

A concrete case helps fix the idea. A model predicting house prices from a
hundred candidate features — square footage, zip code, number of bathrooms,
and dozens of weaker, noisier signals alongside them — will often overfit
badly without a penalty, assigning large coefficients to a handful of those
weaker features simply because they happened to correlate with price in the
training set. L1 regularization applied here tends to zero out most of the
noisy features entirely, leaving a short, interpretable list of the ones that
actually carry signal. L2 applied to the same model keeps every feature in
play but shrinks the noisy ones toward a near-negligible weight, which is
often preferable when even a weak feature might still carry a sliver of
genuine information worth keeping.
