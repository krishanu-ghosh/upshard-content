---
title: Gradient descent
description: The cost function, the gradient as the direction of steepest ascent, and learning as repeatedly stepping the other way.
roadmap: engine-preview
node: gradient-descent
group: how-a-network-learns
video:
  provider: youtube
  id: "IHZwWFHWa-w"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1233
level: beginner
prerequisites: [derivatives, what-is-a-neural-network]
takeaways:
  - Training is minimising a cost function over the weights, not over the input data.
  - The gradient points in the direction of steepest increase, so learning steps along its negative.
  - The step size is a choice with consequences — too large overshoots, too small never arrives.
check_questions:
  - q: What exactly are the variables being optimised?
    a: The weights and biases. The training examples are fixed; the cost is a function of the parameters alone.
  - q: Why does anyone settle for a local minimum?
    a: Because in very high dimensions the local minima that gradient descent actually reaches tend to be good enough, and finding a global one is not tractable.
deeper:
  - label: Why momentum really works
    url: https://distill.pub/2017/momentum/
related_formulas: [gradient, gradient-descent, stochastic-gradient-descent, cross-entropy]
---

## From a fixed network to a learning one

The previous node left a network full of arbitrary numbers. This one adds the
only two things needed to improve them: a number that says how wrong the
network currently is, and a rule for which way to nudge every weight to make
that number smaller.

## Where this sits

Gradient descent says *step along the negative gradient*. It does not say how to
compute that gradient for a network with millions of weights. That is
backpropagation, and it is the next node.
