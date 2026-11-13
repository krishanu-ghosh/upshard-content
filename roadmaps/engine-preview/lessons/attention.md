---
title: Attention
description: Queries, keys and values, scaled dot-product attention, and why dividing by the square root of the dimension is not cosmetic.
roadmap: engine-preview
node: attention
group: transformers
video:
  provider: youtube
  id: "eMlx5fFNoYc"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 1569
level: intermediate
prerequisites: [dot-products, what-is-a-transformer]
takeaways:
  - Every token forms a query, a key and a value, each a learned linear map of its current vector.
  - Attention weights are a softmax over query–key dot products, scaled by the square root of the head dimension.
  - Masking is what keeps a language model from reading tokens it has not generated yet.
check_questions:
  - q: Why divide by the square root of the head dimension before the softmax?
    a: To keep the variance of the dot products from growing with dimension. Without it the softmax saturates and the gradients through it vanish.
  - q: What is the value vector for, as distinct from the key?
    a: The key decides how much attention a token receives; the value is what actually gets added to the attending token's vector.
deeper:
  - label: Attention Is All You Need
    url: https://arxiv.org/abs/1706.03762
related_formulas: [scaled-dot-product-attention, softmax, dot-product]
---

## The mechanism, in one pass

Attention is three linear maps and a weighted average, and almost all of the
difficulty is in keeping track of which vector is playing which role. The
query–key dot product is the same operation from the second node of this
roadmap, now asked a specific question: how relevant is that token to this one?

## Where this sits

This is the last node of the preview. It is also the clearest illustration of
why the formula library exists alongside the roadmaps: the expression behind
this lesson has its own page, with its symbols named, and lessons and formulas
link to each other from both sides.
