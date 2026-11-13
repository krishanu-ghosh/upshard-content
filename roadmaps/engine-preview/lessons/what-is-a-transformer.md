---
title: What is a transformer?
description: Embeddings, the residual stream, and the shape of the computation that turns a prompt into a next-token distribution.
roadmap: engine-preview
node: what-is-a-transformer
group: transformers
video:
  provider: youtube
  id: "wjZofJX0v4M"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1634
level: intermediate
prerequisites: [backpropagation-calculus]
takeaways:
  - Tokens become vectors, and those vectors are repeatedly updated as they pass through the model.
  - Directions in embedding space carry meaning, which is why vector arithmetic on them is sometimes sensible.
  - The final step is a softmax over the vocabulary, and temperature is a knob on that softmax.
check_questions:
  - q: What does the model actually output?
    a: A probability distribution over the whole vocabulary for the next token. Everything else — sampling, temperature, top-p — operates on that distribution.
  - q: What does raising the temperature do?
    a: It flattens the distribution before sampling, making unlikely tokens relatively more likely. Lowering it sharpens towards the argmax.
deeper:
  - label: The Illustrated Transformer
    url: https://jalammar.github.io/illustrated-transformer/
related_formulas: [softmax, temperature-sampling, top-p-sampling, positional-encoding]
---

## A different architecture, the same training story

Nothing in the previous group is discarded here. A transformer is still a stack
of matrix multiplications trained by gradient descent with backpropagation. What
changes is the shape of the stack and, crucially, that each position's vector
can read from every other position's.

## Where this sits

The final node opens the one mechanism this video treats as a black box. It is
the last node of the preview, and it is also where the formula library and the
roadmap meet: scaled dot-product attention is a page in both.
