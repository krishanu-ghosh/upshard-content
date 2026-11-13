---
title: K-nearest neighbors
description: A non-parametric method that classifies or predicts from the k closest training examples — choosing k, scaling sensitivity, and prediction-time cost.
roadmap: machine-learning
node: k-nearest-neighbors
group: supervised-models
video:
  provider: youtube
  id: "HVXime0nQeI"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 330
level: beginner
prerequisites: [logistic-regression]
takeaways:
  - K-nearest neighbors makes no assumption about the underlying relationship between inputs and output; it simply looks at the k closest training examples to a new point and predicts based on them.
  - A small k follows the local data closely and risks overfitting to noise; a large k smooths over local detail and risks underfitting by averaging across points that are not really representative of the new one.
  - Because it relies entirely on distance, k-nearest neighbors is one of the models most sensitive to feature scaling — an unscaled feature with a large range will dominate every distance calculation.
check_questions:
  - q: What does it mean that k-nearest neighbors is "non-parametric"?
    a: It does not fit a fixed set of coefficients to a formula during training the way linear or logistic regression does. Training is essentially just storing the data, and all the real computation happens at prediction time by comparing the new point to the stored examples.
  - q: Why does k-nearest neighbors get slower as the training set grows, in a way linear regression does not?
    a: Predicting a new point requires computing its distance to every single stored training example. Linear regression's prediction cost is fixed once trained, regardless of how large the original training set was, because only the learned coefficients are needed at prediction time.
deeper:
  - label: "IBM Technology — What is the K-Nearest Neighbor Algorithm?"
    url: https://www.youtube.com/watch?v=b6uHw7QW_n4
related_formulas: [euclidean-norm]
---

## Predicting by resemblance

K-nearest neighbors is built on an idea simpler than almost anything else in
this roadmap: to predict a new point, find the k training examples closest to
it and predict based on what they show. For classification, that usually
means a majority vote among the k neighbours' labels; for predicting a
continuous value, it usually means their average. There is no formula fit
during training at all — the "model" is essentially just the stored training
data itself, which is why this approach is called **non-parametric**: it
makes no assumption in advance about the shape of the relationship between
inputs and output, unlike linear regression's assumption that the
relationship is a straight line.

## Choosing k

A small value of k, such as one or three, bases each prediction on only a
handful of the closest points, which follows local detail in the data closely
but is highly sensitive to noise — a single mislabelled or unusual nearby
point can swing the prediction. A large value of k averages over many more
points, smoothing out that noise, but risks washing out genuinely local
structure by including points that are not really representative of the area
around the new one. The right value sits between these extremes and is
typically chosen the same way any other hyperparameter is, covered in
[hyperparameter tuning](/roadmaps/machine-learning/hyperparameter-tuning):
by trying several values and checking which performs best on data the model
was not trained on.

## Why scaling matters more here than almost anywhere else

K-nearest neighbors makes every prediction by computing distances, directly
as covered in
[feature scaling and encoding](/roadmaps/machine-learning/feature-scaling-and-encoding), which makes it one of the models most sensitive to unscaled
features of any covered in this roadmap. A feature measured in the tens of
thousands will dominate a Euclidean distance calculation — see
[Euclidean norm](/formulas/euclidean-norm) for the underlying formula — over
one measured between zero and one, regardless of which feature is actually
more relevant to the prediction. Scaling every feature onto a comparable
range before training is close to mandatory here, in a way it simply is not
for a tree-based model.

## Distance itself is a choice, not a given

"Closest" assumes a specific definition of distance, and the usual default,
**Euclidean distance** (straight-line distance, as the crow flies), is not
the only option. **Manhattan distance** sums the absolute differences along
each dimension instead of the straight-line path between them — named for
the grid of city blocks where you cannot cut diagonally through a building —
and tends to be less sensitive to a single dimension with an extreme outlier
value, since that dimension's contribution is not squared the way it is
inside a Euclidean calculation. Neither metric is universally correct; the
right one depends on what "similar" should actually mean for the specific
features involved, and switching between them is a real, low-cost lever to
try when a model's neighbours do not look sensible on inspection.

## The cost that shows up at prediction time, not training time

Most models do the expensive work during training and make predictions
cheaply afterward. K-nearest neighbors inverts this: "training" is nearly
free, since it amounts to storing the data, but every single prediction
requires computing the new point's distance to every stored training example.
This means prediction cost grows directly with the size of the training set
— a dataset with ten million training examples makes every single prediction
noticeably more expensive than one with ten thousand, which is rarely true of
a model like linear regression, whose prediction cost depends only on the
number of coefficients, fixed once training finishes, and not on how large
the training set originally was.
