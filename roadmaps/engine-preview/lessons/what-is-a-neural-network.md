---
title: What is a neural network?
description: Layers, weights, biases and activations — the whole forward pass, drawn on one handwritten digit.
roadmap: engine-preview
node: what-is-a-neural-network
group: how-a-network-learns
video:
  provider: youtube
  id: "aircAruvnKk"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1120
level: beginner
prerequisites: [matrix-multiplication]
takeaways:
  - A layer is a matrix multiplication, a bias, and a nonlinearity, applied in that order.
  - Weights are what the network learns; the architecture is what you choose before it learns anything.
  - Without the nonlinearity, any stack of layers collapses into a single layer.
check_questions:
  - q: What would happen to a ten-layer network if you removed every activation function?
    a: It would become mathematically identical to a one-layer network, because a composition of linear maps is itself linear.
  - q: What is the bias term for?
    a: It shifts the threshold at which a unit starts responding, so a unit is not forced to pass through the origin.
deeper:
  - label: A visual proof that neural networks can compute any function
    url: http://neuralnetworksanddeeplearning.com/chap4.html
related_formulas: [matrix-multiplication, relu, sigmoid]
---

## The architecture, before any learning

This node is deliberately about the forward pass only. A network that has
learned nothing still computes something, and understanding what it computes —
a chain of matrix multiplications with a squashing function between them — makes
the training story in the next nodes much shorter.

## Where this sits

Everything so far has been fixed. The next node introduces the cost function and
the idea of nudging weights downhill, which is where a network stops being a
formula and starts being a thing that learns.
