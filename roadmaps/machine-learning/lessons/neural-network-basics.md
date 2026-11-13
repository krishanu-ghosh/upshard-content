---
title: Neural network basics
description: The perceptron and multi-layer networks — neurons, weights and biases — and why stacking layers needs non-linearity to beat linear regression.
roadmap: machine-learning
node: neural-network-basics
group: neural-network-foundations
video:
  provider: youtube
  id: "aircAruvnKk"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1120
level: intermediate
prerequisites: [model-selection]
takeaways:
  - A neural network is built from layers of simple units, each computing a weighted sum of its inputs plus a bias, exactly as linear regression does — the network's power comes from stacking many of these and adding non-linearity between layers.
  - Without a non-linear activation function between layers, any number of stacked linear layers collapses mathematically into one single linear layer, no more powerful than ordinary linear regression.
  - A network's weights and biases are its learned parameters, discovered during training by the same kind of optimisation — minimizing a cost function — that fits a linear regression, just applied to vastly more parameters at once.
check_questions:
  - q: What computation does a single neuron actually perform?
    a: It takes its inputs, multiplies each by a learned weight, sums the results together with a learned bias term, and then passes that sum through an activation function — mechanically the same weighted sum linear regression computes, with a non-linear step added afterward.
  - q: Why can't stacking several linear layers alone produce a more powerful model than a single linear layer?
    a: A linear function applied to a linear function is still just another linear function — the stacking collapses mathematically into one equivalent linear transformation. Only inserting a non-linear activation function between the layers lets the stack represent something a single linear layer fundamentally could not.
deeper:
  - label: "StatQuest — The Essential Main Ideas of Neural Networks"
    url: https://www.youtube.com/watch?v=CqOfi41LfDw
related_formulas: [sigmoid, relu]
---

## The model this roadmap has been building toward

Everything in this roadmap's classical-models groups fit a relatively simple,
explicit mathematical form to data — a line, a tree of thresholds, a margin
between classes. A neural network is a different kind of model: instead of
committing in advance to one specific mathematical shape, it is built from
many extremely simple computational units, arranged in layers, whose combined
behaviour can approximate an enormous range of possible relationships between
inputs and outputs — provided it has the right structure and is trained
correctly, both of which the rest of this group covers.

## A single neuron is nothing new

A single neuron's computation is almost exactly what
[linear regression](/roadmaps/machine-learning/linear-regression) already
does: take every input, multiply each by its own learned weight, sum the
results together with a learned bias term added on top. If this were the
entire story, a neural network would be no more powerful than linear
regression, no matter how many neurons were used — which is exactly the
problem the next idea solves.

## Non-linearity is what makes stacking worth anything

Composing one linear function with another linear function produces, after
the algebra is worked through, just another linear function — stacking ten
purely linear layers collapses mathematically into something no more
expressive than a single one. This is why an **activation function** — a
non-linear function such as [sigmoid](/formulas/sigmoid) or
[ReLU](/formulas/relu), covered properly in the next lesson — is applied
after every layer's weighted sum, before the result passes to the next layer.
Inserting that non-linear step between layers is precisely what lets a
multi-layer network represent relationships a single linear layer
structurally cannot, no matter how its weights are set.

## The universal approximation idea

This combination — enough layers, enough neurons per layer, and a non-linear
activation between each one — gives a neural network a property no model
earlier in this roadmap has: with enough capacity, it can approximate
essentially any reasonably well-behaved function connecting inputs to
outputs, a result usually called the universal approximation property. This
does not mean a network automatically finds that approximation in practice —
training it correctly, covered in
[training neural networks](/roadmaps/machine-learning/training-neural-networks), and giving it a sensible architecture are both still required — but it
explains why this family of models can represent patterns, from recognising
images to generating coherent text, that no single linear or even tree-based
model, however large, is structurally capable of representing at all.

## Weights and biases are learned, not designed

Just as a linear regression's coefficients are fit by minimising a cost
function rather than chosen by a person, a neural network's weights and
biases — easily millions or billions of individual numbers in a large
network — are discovered the same way: by defining a cost function measuring
how wrong the network's current predictions are, and adjusting every weight
and bias to reduce that cost. The scale is vastly larger than anything
earlier in this roadmap, but the underlying principle, optimising parameters
against a cost function, is exactly the same idea introduced all the way back
in [linear regression](/roadmaps/machine-learning/linear-regression). How
that optimisation actually happens across so many parameters at once is the
subject of the next two lessons.
