---
title: Logistic regression
description: A classification method despite the name — the sigmoid function, the decision boundary it creates, and the log-loss objective it is trained against.
roadmap: machine-learning
node: logistic-regression
group: supervised-models
video:
  provider: youtube
  id: "yIYKR4sgzI8"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 528
level: beginner
prerequisites: [linear-regression, imbalanced-data]
takeaways:
  - Logistic regression takes the same weighted sum of inputs as linear regression, then passes it through a sigmoid function to squash it into a probability between zero and one.
  - A threshold on that probability — usually 0.5 — turns a continuous probability into a hard classification, and that threshold is itself a choice with consequences, not a fixed rule.
  - It is trained by minimizing log-loss rather than squared error, because squared error on a probability behaves very differently from squared error on a continuous value and gives poor training signal here.
check_questions:
  - q: Why can't linear regression be used directly for a yes/no classification problem?
    a: Linear regression's raw output is an unbounded number that can fall anywhere, including well below zero or above one, which makes no sense interpreted as a probability of belonging to a class.
  - q: What does moving the decision threshold away from 0.5 actually trade off?
    a: Raising the threshold makes the model more conservative about predicting the positive class, reducing false positives at the cost of more false negatives; lowering it does the reverse. The right threshold depends on which kind of mistake costs more in a specific application.
deeper:
  - label: "Gate Smashers — Logistic Regression with a worked example"
    url: https://www.youtube.com/watch?v=r8OjlgWpAI0
  - label: "Visually Explained — Logistic Regression vs Linear Regression"
    url: https://www.youtube.com/watch?v=3bvM3NyMiE0
related_formulas: [sigmoid, cross-entropy]
---

## A classifier wearing a regression's name

Despite its name, logistic regression is a classification method, used to
predict which of two categories something belongs to — whether an email is
spam, whether a transaction is fraudulent, whether a patient has a condition.
The name survives because the model starts exactly like linear regression,
computing a weighted sum of the inputs, and then adds one crucial step that
turns that sum into something usable for classification.

## Why linear regression's raw output cannot be used directly

A plain weighted sum of inputs can be any real number — deeply negative,
wildly positive, anything in between. Interpreted directly as a probability
of belonging to a class, this makes no sense: a probability has to fall
between zero and one, and a raw linear output respects no such bound at all.

## The sigmoid: squashing a number into a probability

The [sigmoid function](/formulas/sigmoid) solves this cleanly. It takes any
real number as input and returns a value strictly between zero and one, with
large positive inputs mapping close to one, large negative inputs mapping
close to zero, and an input of exactly zero mapping to exactly one half.
Logistic regression's full model, then, is: compute the same weighted sum of
inputs that linear regression would, and pass the result through the sigmoid
to get a genuine probability. A threshold, conventionally 0.5, then turns
that probability into a hard yes-or-no classification.

A tiny worked example makes this concrete. Say a model's weighted sum for one
email comes out to `2.2`. Pushed through the sigmoid, that becomes roughly
`0.90` — a 90% estimated chance of being spam, comfortably above the 0.5
threshold, so the email gets flagged. A different email with a weighted sum
of `-1.5` comes out to roughly `0.18`, well under the threshold, so it passes
through. The weighted sum itself carries no intuitive meaning on its own —
only once it is squashed into a probability does a number like `2.2` become
something a person can actually reason about.

## Why the threshold is a choice, not a given

Converting a probability into a decision requires picking a threshold, and
0.5 is only a default, not a law. Raising the threshold makes the model more
conservative — it predicts the positive class only when quite confident,
reducing false positives at the cost of missing more true positives. Lowering
it does the reverse. Which tradeoff is correct depends entirely on the
problem: in a medical screening test, a missed true positive is usually far
costlier than a false alarm, which argues for a lower threshold than 0.5,
while a spam filter with too low a threshold starts flagging real mail,
arguing the other way. This choice connects directly to the precision and
recall tradeoff covered in
[evaluation metrics](/roadmaps/machine-learning/evaluation-metrics).

## Why log-loss, not squared error

Linear regression trains by minimising squared error, and logistic
regression does not — it minimises [cross-entropy](/formulas/cross-entropy),
often called log-loss in this context. The reason is how each objective
behaves near the extremes: squared error treats a prediction of 0.99 when the
true label is 1 as only a marginally better prediction than 0.9, while
log-loss treats a confident wrong answer as catastrophically costly,
penalising a prediction of 0.01 when the true answer is 1 far more severely
than squared error would. This sharper penalty near zero and one gives the
training process a much stronger, more useful gradient to follow when the
model is confidently wrong, which squared error simply does not provide on a
probability output.
