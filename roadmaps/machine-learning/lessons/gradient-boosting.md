---
title: Gradient boosting
description: Training models sequentially, each correcting the ensemble's errors so far, scaled by a learning rate — boosting reduces bias where bagging reduces variance.
roadmap: machine-learning
node: gradient-boosting
group: ensembles-and-kernels
video:
  provider: youtube
  id: "3CC4N4z3GJc"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 952
level: advanced
prerequisites: [bagging-and-random-forests]
takeaways:
  - Gradient boosting builds models sequentially rather than independently, with each new model trained specifically to correct the errors the ensemble so far has made.
  - Each new model's contribution is scaled down by a learning rate before being added to the ensemble, which trades more training iterations for a more stable, less overfit result.
  - Bagging reduces variance by averaging many independent models trained in parallel; boosting reduces bias by building models sequentially, each explicitly targeting what the previous ones got wrong.
check_questions:
  - q: How does gradient boosting's sequential approach differ from bagging's parallel one?
    a: Bagging trains every model independently and in parallel, each on its own random resample, with no model aware of the others. Boosting trains models one after another in sequence, with each new model built specifically to correct the residual errors the ensemble made so far.
  - q: Why does scaling down each new tree's contribution with a learning rate help, rather than just adding its full correction?
    a: Adding each tree's full correction would let the ensemble fit the training data, including its noise, very quickly and aggressively. Scaling each contribution down means more trees are needed to reach the same fit, but the ensemble builds up more gradually and conservatively, which typically generalizes better.
deeper:
  - label: "ritvikmath — Gradient Boosting: Data Science's Silver Bullet"
    url: https://www.youtube.com/watch?v=en2bmeB4QUo
related_formulas: []
---

## Correcting mistakes, one model at a time

Bagging, covered in the previous lesson, trains many models independently and
averages them. Gradient boosting takes a fundamentally different approach: it
builds models one at a time, in sequence, with each new model trained
specifically to correct whatever the ensemble built so far still gets wrong.
The first model makes an initial, usually quite crude prediction. The second
model is trained not on the original targets, but on the **residuals** — the
errors the first model made. The third model is trained on whatever errors
remain after the first two are combined. This continues for as many rounds as
specified, with the final prediction being the sum of every model's
contribution across the whole sequence.

## The learning rate: why not just fix it in one step

Each new tree in the sequence is capable of fitting the current residuals
quite closely, and adding its full correction every time would let the
ensemble fit the training data, including its noise, very aggressively and
very fast. A **learning rate** — typically a small number like 0.1 — scales
down each tree's contribution before it is added to the ensemble, so progress
toward fitting the data happens more gradually, over many more rounds. This
costs more training iterations to reach a comparable fit, but the resulting
ensemble is typically far more stable and generalises better, which is
usually a trade well worth making.

## Bias versus variance, boosting versus bagging

This lesson's companion in the previous group reduces **variance**: many
independent, parallel models, each a little different due to resampling,
average out each other's noise. Gradient boosting instead reduces **bias**:
a single weak model, incapable on its own of capturing the full pattern in
the data, is improved sequentially by each new model explicitly targeting
what remains unexplained, building toward a far more accurate fit than any
one of the individual trees could achieve alone. These are genuinely
different mechanisms addressing genuinely different kinds of error, which is
worth remembering rather than treating "ensemble method" as one
undifferentiated category — the right choice between them depends on whether
a single simple model is systematically missing the pattern (favouring
boosting) or is unstable and noisy but roughly on target (favouring bagging).

## Two more knobs beyond the learning rate

The learning rate is not the only control on how aggressively the ensemble
fits the training data. Limiting each individual tree's depth keeps every
single correction weak and simple, which forces the ensemble to rely on
having many trees rather than a few powerful ones — a shallower tree is less
able to memorise noise specific to one training set. Capping the total
number of trees has the same spirit from a different angle: stop adding
corrections once a held-out validation set's error stops improving, a
technique usually called **early stopping**, rather than training a fixed
number of rounds regardless of whether later ones are still helping. Both
controls exist for the same underlying reason as the learning rate itself —
gradient boosting is powerful enough to fit training data almost perfectly
given enough rounds, so something has to hold it back from doing so at the
expense of generalising to new data.

## XGBoost and LightGBM: gradient boosting, engineered

The specific algorithm described above is often implemented today through
libraries like XGBoost and LightGBM, which add substantial engineering on top
of the core idea — more efficient handling of missing values, built-in
regularization to control overfitting directly, and considerably faster
training on large datasets than a naive implementation would achieve. These
names are worth recognising, because in practice "gradient boosting" very
often means one of these specific, heavily optimised implementations rather
than a from-scratch version of the algorithm, even though the underlying
mechanism — sequential correction of residuals, scaled by a learning rate —
is exactly what is described above in either case.
