---
title: Derivatives
description: The derivative as a rate of change you can see, and the chain rule as the reason deep networks are trainable at all.
roadmap: engine-preview
node: derivatives
group: the-maths-underneath
video:
  provider: youtube
  id: "WUvTyaaNkzM"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1024
level: beginner
prerequisites: [vectors]
takeaways:
  - A derivative is the best linear approximation to a function at a point — a slope, not an instant.
  - Rules like the product and chain rules are statements about how small changes combine, and they can be read off a picture.
  - In several variables the same idea becomes a gradient — one slope per input, collected into a vector.
check_questions:
  - q: What does "instantaneous rate of change" actually mean if nothing moves in an instant?
    a: It is the limit of average rates over shrinking intervals. The value is a property of the limit, not of any single moment.
  - q: Why does the chain rule matter more than any other rule here?
    a: Because a network is a composition of functions. Training it means differentiating a composition, and the chain rule is the only tool that does that.
deeper:
  - label: Essence of calculus — the full series
    url: https://www.youtube.com/playlist?list=PLZHQObOWTQDMsr9K-rj53DwVRMYO3t5Yr
related_formulas: [derivative-definition, chain-rule, gradient]
---

## The one calculus idea you cannot skip

The maths a network needs from calculus is narrow: a derivative is a slope, a
gradient is a vector of slopes, and the chain rule lets you differentiate a
function built out of other functions. That last one is the whole of
backpropagation, two nodes from here.

## Where this sits

The path forks here. One branch goes through the network's architecture, the
other through its training signal, and they meet at gradient descent — which
needs the shape of the network from one side and the derivative from the other.
