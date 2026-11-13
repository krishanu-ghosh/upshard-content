---
title: Backpropagation
description: How gradients of the loss with respect to every weight are computed efficiently via the chain rule, propagating error backward through the network.
roadmap: machine-learning
node: backpropagation
group: neural-network-foundations
video:
  provider: youtube
  id: "Ilg3gGewQ5U"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 767
level: advanced
prerequisites: [activation-functions]
takeaways:
  - Backpropagation computes how much each individual weight in a network contributed to the final error, working backward from the output layer to the input layer using the chain rule.
  - It is an efficient algorithm for computing a gradient, not a separate learning rule — the actual weight updates still come from gradient descent, exactly as in linear regression, just applied to vastly more parameters.
  - Error that originates at the output layer has to pass back through every layer in between, and how cleanly that signal survives the trip depends directly on the activation functions used in those layers.
check_questions:
  - q: What problem does backpropagation actually solve?
    a: Efficiently computing how much each individual weight in a possibly huge network should change to reduce the overall error, without having to separately re-evaluate the entire network once per weight — which the chain rule makes possible by computing these contributions layer by layer, working backward from the output.
  - q: Is backpropagation itself the learning algorithm, or something else?
    a: It computes the gradient — how the error changes with respect to every weight — but the actual update to each weight is still performed by gradient descent, exactly the same optimisation idea used to fit a linear regression, just applied here across every weight in the network at once.
deeper:
  - label: "3Blue1Brown — Backpropagation calculus (Deep Learning Chapter 4)"
    url: https://www.youtube.com/watch?v=tIeHLnjs5U8
  - label: "StatQuest — Backpropagation Main Ideas, with worked numbers"
    url: https://www.youtube.com/watch?v=IN2XmBhILt4
related_formulas: [chain-rule, gradient]
---

## The problem: whose fault was the error?

A network with many layers and millions of weights produces one final error
at its output. Training requires knowing how much each individual weight,
scattered across every layer, contributed to that error — information needed
to know which direction and how much to adjust each one. Checking this one
weight at a time, by nudging it slightly and re-running the entire network to
see what changes, would be prohibitively slow for anything beyond a tiny
network. Backpropagation is the algorithm that computes this efficiently for
every weight at once.

## The chain rule, applied layer by layer

The mathematical tool underneath backpropagation is the
[chain rule](/formulas/chain-rule): if a change in one quantity affects a
second quantity, which in turn affects a third, the chain rule lets you
compute how the first quantity ultimately affects the third by multiplying
the individual effects along the chain. A neural network is exactly this kind
of chain, stretched across many layers — a weight in an early layer affects
that layer's output, which affects the next layer's output, and so on until
it finally affects the error at the very end. Backpropagation applies the
chain rule systematically, working backward from the output layer toward the
input layer, computing at each layer how the error depends on that layer's
weights, using quantities already computed for the layer just after it. This
backward, layer-by-layer reuse is what makes the whole computation efficient
rather than requiring a separate pass for every individual weight.

## It computes the gradient; it does not update anything itself

A point worth being precise about: backpropagation is not a separate learning
algorithm standing apart from everything earlier in this roadmap. It computes
the **gradient** — exactly the same kind of quantity
[gradient descent](/formulas/gradient-descent) already uses to fit a linear
regression, here applied across every weight in a much larger network at
once. Once that gradient is known for every weight, the actual update — move
each weight a small step in the direction that reduces the error — is
performed by gradient descent, covered properly in
[training neural networks](/roadmaps/machine-learning/training-neural-networks), exactly the same optimisation idea introduced all the way back
at the very start of this roadmap's supervised-models group.

## Why the activation function choice from the last lesson matters here

The error signal computed by backpropagation has to travel backward through
every layer's activation function on its way from the output to the input.
At each layer, the signal gets multiplied by that activation function's local
gradient. That is precisely why
[the previous lesson's](/roadmaps/machine-learning/activation-functions)
coverage of vanishing gradients matters directly here: an activation function
whose gradient shrinks toward zero for large inputs causes the backward error
signal to shrink at every layer it passes through. By the time it reaches an
early layer in a genuinely deep network, it can become too small to produce
a meaningful weight update there at all. Backpropagation is the mechanism
carrying the signal backward — whether that signal survives the trip intact
depends on the activation functions it has to pass through along the way.
