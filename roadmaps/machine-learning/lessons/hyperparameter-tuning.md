---
title: Hyperparameter tuning
description: Parameters versus hyperparameters; grid search, random search and Bayesian optimization — and the risk of overfitting to the validation set itself.
roadmap: machine-learning
node: hyperparameter-tuning
group: evaluating-models
video:
  provider: youtube
  id: "xRhPwQdNMss"
  start: 0
  end: null
creator:
  name: Prof. Ryan Ahmed
  channel_url: https://www.youtube.com/@professor-ryanahmed
duration_seconds: 482
level: intermediate
prerequisites: [cross-validation, support-vector-machines]
takeaways:
  - A parameter is learned automatically from data during training, like a regression coefficient; a hyperparameter is set before training even starts, like the number of trees in a forest or the strength of a regularization penalty.
  - Grid search exhaustively tries every combination from a specified set of values; random search samples combinations at random instead, which often finds a comparably good configuration far faster.
  - Repeatedly checking different hyperparameter configurations against the same validation data can itself overfit to that validation set, which is exactly why a final, untouched test set still matters after tuning is done.
check_questions:
  - q: What is the difference between a parameter and a hyperparameter?
    a: A parameter is learned automatically during training from the data, such as a regression coefficient or a neural network's weights. A hyperparameter is set before training begins and is not learned from the data directly — the number of neighbours in k-nearest neighbors, or the depth limit on a decision tree.
  - q: Why can random search often outperform grid search despite trying fewer combinations?
    a: Grid search spends equal effort on every hyperparameter, including ones that barely affect performance, because it tries every combination of the full grid. Random search samples more varied values along the hyperparameters that actually matter, which in practice often finds a comparably good or better configuration in a fraction of the evaluations.
deeper:
  - label: "Pankaj Kumar Porwal — Parameters vs Hyperparameters"
    url: https://www.youtube.com/watch?v=32tNAhQ8x7M
  - label: "NeuralNine — Hyperparameter Tuning Explained"
    url: https://www.youtube.com/watch?v=Mxx6imqkkSA
related_formulas: []
---

## A distinction worth being precise about

A **parameter** is something a model learns automatically from the training
data — a linear regression's coefficients, a neural network's weights. A
**hyperparameter** is something set before training even begins, and the
training process does not adjust it on its own — the number of neighbours in
[k-nearest neighbors](/roadmaps/machine-learning/k-nearest-neighbors), the
depth limit on a [decision tree](/roadmaps/machine-learning/decision-trees),
the strength of a
[regularization](/roadmaps/machine-learning/regularization) penalty, the
number of trees in a
[random forest](/roadmaps/machine-learning/bagging-and-random-forests). Every
model covered so far in this roadmap has both kinds of setting, and this
lesson is specifically about searching for good hyperparameter values, since
nothing in ordinary training finds them automatically.

## Grid search: try everything

**Grid search** specifies a set of candidate values for each hyperparameter
and tries every possible combination of them, training and evaluating a
separate model for each one. This is exhaustive and guaranteed to check every
combination within the specified grid, but the number of combinations grows
multiplicatively with the number of hyperparameters being tuned — three
hyperparameters with five candidate values each already means a hundred and
twenty-five separate models to train, and adding a fourth hyperparameter
multiplies that further.

## Random search: sample instead of enumerate

**Random search** instead samples a fixed number of random combinations from
the same space of possible values, rather than trying every combination
exhaustively. This sounds like it should perform worse, and in practice it
frequently performs comparably or even better for a given computational
budget, because not every hyperparameter affects performance equally — grid
search spends effort testing every value of a hyperparameter that barely
matters just as thoroughly as one that matters a great deal, while random
search's sampling tends to explore a wider range of values along whichever
hyperparameters actually turn out to matter most.

## Bayesian optimization: learning where to look next

**Bayesian optimization** goes a step further, using the results of
previously tried configurations to intelligently choose which configuration
to try next, rather than following a fixed grid or sampling purely at random.
It builds a probabilistic model of how performance relates to hyperparameter
values based on what has been tried so far, and uses that model to pick the
next, most promising point to evaluate. This tends to find good
configurations in far fewer total evaluations than either grid or random
search, at the cost of being more complex to set up and reason about.

## The risk hiding inside this whole process

Every hyperparameter search evaluates each candidate configuration against
the same validation data, repeatedly, often across dozens or hundreds of
configurations. This is a genuine and easily overlooked risk: checking enough
different configurations against the same validation set can itself produce
a configuration that happens to fit that particular validation set well,
in much the same way a model can overfit to its training data, covered in
[generalization and overfitting](/roadmaps/machine-learning/generalization-and-overfitting). This is exactly why the discipline of a final, completely
untouched test set still matters after all the tuning is done — it is the
only honest check on whether the chosen configuration generalises, or merely
happened to fit the validation data the search process spent so much effort
optimising against.
