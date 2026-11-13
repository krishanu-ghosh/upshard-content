---
title: Hyperparameter tuning
description: Grid search, random search, and Bayesian optimization — systematic ways to search a config space instead of tuning by hand.
roadmap: mlops
node: hyperparameter-tuning
group: experimentation
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
prerequisites: [model-registries]
takeaways:
  - Hyperparameters are settings chosen before training starts (learning rate, batch size, tree depth, and similar) that the training process itself cannot learn, so they have to be searched over separately.
  - Grid search and random search both explore the configuration space blindly, while Bayesian optimization uses the results of earlier trials to decide where to try next, which finds good regions with far fewer runs.
  - A search over a high-dimensional space grows exponentially under grid search, which is exactly why random search and smarter methods tend to outperform it at a fixed compute budget.
check_questions:
  - q: What makes a hyperparameter different from a regular model parameter?
    a: A hyperparameter is set before training begins and controls the training process itself, so gradient descent or whatever learning algorithm is used cannot learn its value the way it learns model weights.
  - q: Why does grid search become impractical as you add more hyperparameters to tune?
    a: Because the number of combinations it must evaluate grows multiplicatively with each added hyperparameter and each added value per hyperparameter, so the search space explodes exponentially.
  - q: How does Bayesian optimization choose which configuration to try next, in contrast to grid or random search?
    a: It builds a probabilistic model of how past trials' configurations relate to their results, and uses that model to pick the next configuration most likely to improve on what's been tried, rather than picking blindly.
deeper:
  - label: "Hyperparameter Optimization: This Tutorial Is All You Need — Abhishek Thakur"
    url: https://www.youtube.com/watch?v=5nYqK-HaoKY
related_formulas: []
---

## The knobs training can't set itself

Every model has two kinds of numbers attached to it. There are the
parameters — weights and biases — that training actually learns by
minimizing a loss function. And there are hyperparameters: the learning
rate, the batch size, the number of layers, the maximum depth of a
decision tree, the regularization strength. These are set *before*
training starts, and training has no mechanism to adjust them itself —
gradient descent optimizes weights against a loss, but it has no gradient
with respect to "how many layers should this network have." Get the
hyperparameters wrong and no amount of training fixes it: a learning rate
that's too high makes loss diverge or oscillate no matter how long you
train; a tree depth that's too large overfits regardless of how much data
you feed it. Finding good hyperparameters is a search problem layered on
top of training, not something training solves on its own, and that's
what makes it a distinct, systematic practice rather than something you
eyeball once.

## Why hand-tuning breaks down

The naive approach — try a learning rate, look at the result, adjust by
feel, repeat — works for one or two hyperparameters when you have a lot
of intuition about the problem. It stops working once you have five or
six interacting hyperparameters, because their effects aren't independent:
a learning rate that's great at batch size 32 might be unstable at batch
size 256, so tuning one at a time and holding the rest fixed can land you
in a configuration that's locally fine but far from the actual best
combination. Hand-tuning also doesn't scale across people or time — it
depends on one person's accumulated intuition, which isn't reproducible,
documented, or transferable to the next model.

## Grid search and random search

Grid search is the direct, brute-force fix: pick a discrete set of values
for each hyperparameter (say, learning rate in {0.1, 0.01, 0.001} and
batch size in {16, 32, 64, 128}), and run training for every combination —
here, 3 × 4 = 12 runs. It's exhaustive within the grid you defined, which
is also its flaw: the cost grows multiplicatively with every hyperparameter
and every value you add. Tune five hyperparameters with 5 values each and
you're at 5⁵ = 3,125 runs — most of them wasted exploring regions that
were never going to be competitive.

Random search fixes the scaling problem by sampling configurations
randomly from each hyperparameter's range instead of exhaustively covering
a grid, for a fixed number of trials you choose up front. This sounds
worse, but empirically it tends to beat grid search at the same budget,
for a specific reason: most hyperparameters don't matter equally. If
learning rate matters a lot and a particular regularization term barely
matters, grid search wastes many of its runs testing every regularization
value against every learning rate value, while random search's trials
naturally cover more distinct learning-rate values for the same total
run count, because it isn't locked into a fixed grid structure.

## Bayesian optimization

Both grid and random search share a weakness: every trial is chosen
independently of what previous trials revealed. Bayesian optimization
removes that blindness. After each trial, it updates a probabilistic
model (commonly a Gaussian process) of how the hyperparameters relate to
the outcome, and uses that model to pick the next configuration — favoring
regions that look promising, while still occasionally testing uncertain
regions that might hide something better. Concretely: if learning rates
near 0.01 have consistently scored well and 0.1 has consistently scored
poorly, the next trial is likely to probe near 0.01 or 0.005, not 0.1
again. This lets it find strong configurations in a fraction of the
trials grid search would need, at the cost of being sequential — each
trial's choice depends on the last, so it parallelizes less naturally than
random search, which can fire off all its trials at once.

## Why this matters going forward

Hyperparameter tuning only pays off because of the two lessons before it
in this group: it needs experiment tracking to record every trial's
config and result, and it needs a model registry to decide which of the
many configurations tried is actually worth registering as a candidate.
Without tracking, each tuning run would be exploring blind with no memory
of what it already tried — which is the exact problem Bayesian
optimization is built to exploit, and the exact problem it can't solve if
the history isn't being kept in the first place.
