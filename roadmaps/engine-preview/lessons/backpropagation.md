---
title: Backpropagation
description: How a single training example votes on every weight in the network, and why the votes are averaged rather than applied.
roadmap: engine-preview
node: backpropagation
group: how-a-network-learns
video:
  provider: youtube
  id: "Ilg3gGewQ5U"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 767
level: intermediate
prerequisites: [gradient-descent]
takeaways:
  - Backpropagation computes the gradient; gradient descent is what then uses it.
  - Each training example proposes a change to every weight, and the batch's proposals are averaged.
  - Influence flows backwards — a weight matters in proportion to the activation feeding it and the error above it.
check_questions:
  - q: What is the difference between backpropagation and gradient descent?
    a: Backpropagation is the algorithm that computes the gradient efficiently. Gradient descent is the rule that takes a step once you have it.
  - q: Why use mini-batches rather than the whole training set?
    a: Because a noisy gradient from a small sample is far cheaper per step and, in practice, converges faster than the exact gradient computed rarely.
deeper:
  - label: Calculus on computational graphs, written out
    url: https://colah.github.io/posts/2015-08-Backprop/
related_formulas: [chain-rule, gradient, stochastic-gradient-descent]
---

## The algorithm, before the notation

This node is the intuition pass, and it is worth taking it on its own terms
before meeting the symbols in the next one. The claim is simple: for one
training example you can say, for every weight in the network, whether it should
go up or down and roughly how strongly it wants to.

## Where this sits

The next node writes the same algorithm in the language of partial derivatives.
Nothing new happens there — it is this argument, made precise enough to
implement.
