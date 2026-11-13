---
title: Attention variants for serving
description: Multi-head, multi-query and grouped-query attention — why reducing the number of KV heads became a serving-performance decision, not just a modeling one.
roadmap: inference-engineering
node: attention-variants-for-serving
group: inside-the-forward-pass
video:
  provider: youtube
  id: "TvRqn9sASvU"
  start: 0
  end: null
creator:
  name: TechWithSid
  channel_url: https://www.youtube.com/@TechWithSid-k9j
duration_seconds: 1057
level: advanced
prerequisites: [kv-cache-mechanics]
takeaways:
  - Multi-head attention gives every attention head its own key and value projections, which is expressive but means the KV cache has to store a full set of keys and values per head.
  - Multi-query attention shares a single key and value projection across all heads, shrinking the KV cache dramatically at some cost to model quality.
  - Grouped-query attention splits the difference, sharing key and value projections across small groups of heads rather than all of them or none, recovering most of the quality while keeping most of the memory saving.
check_questions:
  - q: Why does multi-query attention shrink the KV cache so much compared to multi-head attention?
    a: Multi-head attention stores a separate key and value vector per attention head, so cache size scales with the number of heads. Multi-query attention uses one shared key and value projection across every head, so the cache only has to store one head's worth regardless of how many heads the model actually has.
  - q: What tradeoff does grouped-query attention make relative to the other two?
    a: It shares key and value projections across small groups of heads rather than across all heads (multi-query) or none (multi-head), landing on a middle ground that keeps most of multi-query's memory saving while recovering most of multi-head's modeling quality.
deeper:
  - label: "Vizuara — Multi-Query Attention Explained"
    url: https://www.youtube.com/watch?v=Z6B51Odtn-Y
  - label: "Vizuara — Understand Grouped Query Attention (GQA)"
    url: https://www.youtube.com/watch?v=kx3rETIxo4Q
related_formulas: [scaled-dot-product-attention, kv-cache-size]
---

## A modeling choice that turned into a serving decision

Attention, covered in depth in Upshard's AI Engineer roadmap, is usually
presented as a modeling-quality question: how should a model combine
information across a sequence. At serving time, a different question turns
out to matter just as much: how many separate key and value vectors does the
attention mechanism require you to cache? The answer depends entirely on
which variant of attention a model uses, and it directly determines how
large the [KV cache](/roadmaps/inference-engineering/kv-cache-mechanics)
becomes — which is why this is covered here, in a roadmap about serving,
rather than only as a modeling detail.

## Multi-head attention: one key and value set per head

Standard **multi-head attention (MHA)** gives every attention head its own
independent key and value projections. This is the most expressive version —
each head can specialise in attending to different kinds of information — and
it is also the most expensive to cache, because the KV cache has to store a
complete set of key and value vectors for every single head, at every layer.

## Multi-query attention: one shared set for every head

**Multi-query attention (MQA)** shares a single key and value projection
across every attention head, while still giving each head its own separate
query projection. Because only one head's worth of key and value vectors
ever needs to be cached, regardless of how many heads the model has, this
shrinks the KV cache dramatically — often by many multiples. The cost is a
real, measurable hit to model quality, since every head is now forced to
attend using the same shared keys and values rather than its own.

## Grouped-query attention: a middle ground

**Grouped-query attention (GQA)** splits attention heads into a small number
of groups, with each group sharing one key and value projection rather than
every head sharing one (MQA) or every head having its own (MHA). This
recovers most of MHA's modeling quality — enough heads retain some
independence that the quality loss is far smaller than MQA's — while still
capturing most of MQA's cache savings, since the number of distinct key/value
sets is now a small, chosen number of groups rather than the full head count.
This specific tradeoff point is why GQA has become the default choice across
most current production language models: it buys back almost all of the
memory saving that makes serving affordable at long context and high
concurrency, at a quality cost small enough to be worth it.

## Why this is a serving decision, not just a modeling one

Whether a model uses MHA, MQA or GQA is decided during training and cannot
be changed afterward by anyone serving it — but it directly determines, for
any model you choose to self-host, how much KV cache memory a given context
length and batch size will actually require, per
[KV cache size](/formulas/kv-cache-size). Two models of identical parameter
count can have wildly different memory footprints at serving time purely
based on this one architectural choice, which makes it a genuine factor in
[choosing hardware](/roadmaps/inference-engineering/choosing-hardware) and in
estimating how much concurrent traffic a given GPU can actually sustain.
