---
title: Linear regression
description: Fitting a line to predict a continuous value — the cost function, fitting by minimizing squared error, and interpreting the resulting coefficients.
roadmap: machine-learning
node: linear-regression
group: supervised-models
video:
  provider: youtube
  id: "zUQr6HAAKp4"
  start: 0
  end: null
creator:
  name: Gate Smashers
  channel_url: https://www.youtube.com/@GateSmashers
duration_seconds: 661
level: beginner
prerequisites: [generalization-and-overfitting, feature-scaling-and-encoding]
takeaways:
  - Linear regression fits a straight line (or hyperplane, with more than one input) by finding the coefficients that minimize the sum of squared differences between predictions and actual values.
  - Each coefficient has a direct interpretation — holding every other input fixed, it is the change in the predicted output for a one-unit change in that input.
  - The normal equation solves for the optimal coefficients directly in one step for small problems; gradient descent finds them iteratively and scales to problems too large for the direct solution.
check_questions:
  - q: What does a linear regression coefficient actually mean?
    a: Holding every other input variable fixed, it is the amount the predicted output changes for a one-unit increase in that specific input. A coefficient of 3 on "years of experience" means one more year predicts three more units of the output, all else equal.
  - q: Why minimize the sum of squared errors rather than the sum of plain errors?
    a: Plain errors of opposite sign would cancel out, letting a line that is wrong in both directions appear to fit well on average. Squaring makes every error positive and penalizes large errors disproportionately more than small ones, which is usually the sensible behaviour.
deeper:
  - label: "StatQuest — Linear Regression, Clearly Explained"
    url: https://www.youtube.com/watch?v=7ArmBVF2dCs
related_formulas: [gradient-descent, stochastic-gradient-descent]
---

## The simplest model that still teaches the whole idea

Linear regression predicts a continuous number as a weighted sum of the
inputs plus a constant: a predicted house price as some amount per square
foot, plus some amount per bedroom, plus a base value, each weighted by a
coefficient the training process finds. It is usually the first model anyone
learns, not because it is the most powerful but because nearly every idea
that recurs through the rest of this roadmap — a cost function, fitting by
optimisation, the risk of overfitting, interpreting what a model actually
learned — shows up here in its simplest possible form.

## The cost function: what "best fit" means

A line can be drawn through any scatter of points; the question is which line
is best. Linear regression answers this with **mean squared error**: for
every training point, compute the difference between the line's prediction
and the actual value, square that difference, and average over all points.
Squaring matters for two reasons — it makes every error positive, so errors
in opposite directions cannot cancel out and hide a genuinely bad fit, and it
penalises a large error far more heavily than several small ones, which
usually matches what "a bad prediction" should mean in practice.

Fitting the model means finding the coefficients that make this quantity as
small as possible across the entire training set.

## Two ways to find the minimum

The **normal equation** solves for the optimal coefficients directly in one
closed-form calculation, which works well when the number of input features
is small. As the number of features grows, that direct calculation becomes
too expensive, and **gradient descent** — covered in depth on the
[gradient descent](/formulas/gradient-descent) formula page — instead finds
the minimum iteratively: start from an arbitrary set of coefficients, compute
which direction reduces the error fastest, take a small step in that
direction, and repeat until the error stops improving meaningfully. Both
methods arrive at the same answer for linear regression specifically;
gradient descent is the one that generalises to the vastly more complex
models, including every neural network, covered later in this roadmap.

## Reading the coefficients

Once fit, each coefficient has a direct, literal meaning: holding every other
input fixed, it is the change in the predicted output for a one-unit increase
in that specific input. A coefficient of 3 on "years of experience" means one
additional year of experience predicts three more units of whatever is being
predicted, with every other input held constant. This interpretability is a
genuine advantage linear regression retains over far more powerful models —
a stakeholder can be shown exactly what the model concluded about each input,
not merely how accurate its predictions turn out to be.

## Where it breaks down

Linear regression assumes the relationship between inputs and output is
genuinely linear, and it is not a safe assumption everywhere — a line is a
poor fit to a relationship that curves, saturates, or interacts in ways a
straight weighted sum cannot represent. It is also sensitive to outliers,
because one far-flung point contributes a squared error disproportionate to
how much it should actually influence the fit. These limits are exactly what
motivate the
[regularization](/roadmaps/machine-learning/regularization) lesson that
follows, and eventually the move toward models, including
[decision trees](/roadmaps/machine-learning/decision-trees) and neural
networks, capable of representing relationships a straight line cannot.
