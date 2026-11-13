---
title: Training neural networks
description: The practical training loop — forward pass, loss, backward pass, optimizer step, batch size, epochs — and batch normalization and dropout as stabilizers.
roadmap: machine-learning
node: training-neural-networks
group: neural-network-foundations
video:
  provider: youtube
  id: "IHZwWFHWa-w"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1233
level: advanced
prerequisites: [backpropagation]
takeaways:
  - Training repeats one loop many times — compute predictions (forward pass), measure error (loss), compute gradients (backward pass via backpropagation), and update every weight a small step (optimizer step) — until performance stops improving meaningfully.
  - A batch is a subset of training data used for one weight-update step rather than the full dataset at once, which trades a noisier gradient estimate for the ability to update weights far more often per pass through the data.
  - Batch normalization stabilizes training by keeping each layer's inputs in a consistent range, and dropout fights overfitting by randomly disabling neurons during training, which prevents the network from relying too heavily on any single one.
check_questions:
  - q: Why use batches rather than computing the gradient over the entire training set at once?
    a: Computing the gradient over the whole dataset for every single update is accurate but extremely slow when the dataset is large, since one weight update requires a full pass through all the data. Using a smaller batch gives a noisier but much cheaper gradient estimate, allowing far more updates to happen in the same amount of time.
  - q: What problem does dropout address, and how?
    a: It addresses overfitting — a network relying too heavily on a small number of specific neurons to make its predictions, which generalizes poorly. During training, dropout randomly disables a fraction of neurons on each pass, forcing the network to spread what it learns across many neurons rather than depending heavily on any particular one.
deeper:
  - label: "StatQuest — Gradient Descent, Step-by-Step"
    url: https://www.youtube.com/watch?v=sDv4f4s2SB8
  - label: "deeplizard — Batch Normalization explained"
    url: https://www.youtube.com/watch?v=dXB-KQYkzNU
related_formulas: [gradient-descent, stochastic-gradient-descent, momentum, rmsprop, adam, dropout, batch-normalization]
---

## The loop underneath every neural network

Training a neural network, regardless of its size or architecture, repeats
one loop: a **forward pass** computes the network's current predictions for a
batch of training examples. The **loss** measures how wrong those predictions
are, exactly as covered in
[evaluation metrics](/roadmaps/machine-learning/evaluation-metrics). The
**backward pass** — backpropagation, covered in the previous lesson —
computes the gradient of that loss with respect to every weight in the
network. The **optimizer step** then uses that gradient to actually update
every weight, moving each one a small distance in the direction that reduces
the loss. This cycle repeats, over and over, across the entire training
dataset, for as many **epochs** — full passes through the data — as it takes
for performance to stop meaningfully improving.

## Batches: trading accuracy for speed

Computing the gradient using the entire training set for every single update
is accurate, but prohibitively slow once a dataset is large — one update
would require a full pass through all the data. Using a smaller **batch** of
training examples for each update gives a noisier estimate of the true
gradient, but allows updates to happen far more frequently for the same
amount of total computation. **Batch size** is itself a setting that trades
off these two concerns — a larger batch gives a more accurate but more
expensive gradient estimate per update; a smaller batch is noisier but allows
many more updates within the same time budget. Training on batches of a size
smaller than the full dataset but larger than a single example is usually
called mini-batch gradient descent, and it is the practical default nearly
everywhere.

## Optimizers: smarter than a plain fixed step

Plain gradient descent moves every weight by the gradient scaled by a fixed
**learning rate**. In practice, several refinements consistently outperform
this plain version. **Momentum** accumulates a running average of recent
gradients, which smooths out the update and helps push through small bumps
and flat regions in the loss landscape that would otherwise slow plain
gradient descent down considerably. **RMSprop** and **Adam** go further,
adapting the effective learning rate separately for every individual weight
based on the recent history of its own gradients — a weight whose gradient
has been consistently small gets a relatively larger effective step, and one
whose gradient has been large and erratic gets a relatively smaller one. Adam
combines the ideas behind momentum and RMSprop together and is, in practice,
the most common default optimizer across modern neural network training.

## Batch normalization: keeping every layer's inputs well-behaved

As training proceeds and earlier layers' weights change, the distribution of
values flowing into a later layer can shift considerably from batch to batch
— a problem that compounds as networks get deeper. **Batch normalization**
addresses this directly by normalising each layer's inputs within every
batch, keeping them in a consistent, well-behaved range throughout training.
This generally makes training both faster and more stable, and it has become
close to a standard component in many modern network architectures.

## Dropout: fighting overfitting by removing redundancy on purpose

**Dropout** addresses overfitting specifically by randomly disabling a
fraction of neurons — commonly somewhere around twenty to fifty percent —
during each training pass, forcing the remaining active neurons to produce
a sensible result without being able to rely on any specific set of other
neurons always being present to help. This prevents the network from
developing an over-reliance on any small group of neurons, pushing it instead
toward representations that are more distributed and less fragile, which
tends to generalise to new data considerably better. Dropout is active only during
training; once training is complete, every neuron is used during actual
prediction.
