---
title: Backpropagation calculus
description: The same algorithm in symbols — partial derivatives, the chain rule, and where each term in the expression comes from.
roadmap: engine-preview
node: backpropagation-calculus
group: how-a-network-learns
video:
  provider: youtube
  id: "tIeHLnjs5U8"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 617
level: intermediate
prerequisites: [backpropagation, derivatives]
takeaways:
  - Every term in the backward pass is a chain-rule factor with a concrete meaning in the network.
  - The derivative with respect to a weight is the activation feeding it times the error signal above it.
  - The same expression generalises from one unit per layer to many by replacing numbers with indices.
check_questions:
  - q: Where does the activation of the previous layer appear in the weight's derivative, and why?
    a: As a multiplying factor. A weight attached to a near-zero activation has almost no effect on the output, so its derivative is small.
  - q: What does moving from one neuron per layer to many change?
    a: Only the bookkeeping. Each quantity gains an index and the sums run over the units of the next layer; the chain rule is unchanged.
deeper:
  - label: The matrix calculus you need for deep learning
    url: https://explained.ai/matrix-calculus/
related_formulas: [chain-rule, jacobian, gradient]
---

## The notation is the point

The previous node argued that influence flows backwards. This one writes that
argument down. It is worth watching with a pen, because the expression for a
single weight's derivative is short enough to reconstruct from memory once, and
reconstructing it once is what makes the general case readable.

## Where this sits

That completes the training story for an ordinary feedforward network. The last
group swaps the architecture for the one language models actually use, and the
training story carries over unchanged.
