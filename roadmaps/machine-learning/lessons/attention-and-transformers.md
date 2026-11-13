---
title: Attention and transformers
description: Why attention let models look at any part of a sequence directly instead of a single bottleneck hidden state, and the architecture built entirely on it.
roadmap: machine-learning
node: attention-and-transformers
group: deep-learning-frontiers
video:
  provider: youtube
  id: "ZXiruGOCn9s"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 351
level: advanced
prerequisites: [recurrent-neural-networks]
takeaways:
  - Attention lets a model look directly at any earlier part of a sequence when producing each output, rather than relying on a single fixed-size hidden state to carry the entire history forward, as an RNN does.
  - A transformer is an architecture built entirely on attention, with no recurrence at all, which lets every position in a sequence be processed in parallel rather than one step at a time.
  - This lesson is deliberately a brief bridge rather than a deep dive — the full mechanics behind modern language models, including tokenization and the specific attention computation they use, are covered in Upshard's separate AI Engineer roadmap.
check_questions:
  - q: What specific limitation of RNNs does attention directly address?
    a: An RNN compresses its entire view of everything seen so far into one fixed-size hidden state, which becomes a bottleneck for long sequences — a lot has to be squeezed into a vector of fixed size. Attention instead lets a model look directly back at any specific earlier position when it needs to, rather than relying entirely on that one compressed summary.
  - q: Why can a transformer process a sequence in parallel, when an RNN cannot?
    a: An RNN must process a sequence one step at a time, because each step's hidden state depends on the step before it. A transformer has no such step-by-step dependency — it looks at the whole sequence through attention at once, which is exactly what makes it far more parallelizable, and therefore faster to train, on modern hardware.
deeper:
  - label: "StatQuest — Attention for Neural Networks, Clearly Explained"
    url: https://www.youtube.com/watch?v=PSs6nxngL6k
  - label: "3Blue1Brown — Attention in transformers, step by step"
    url: https://www.youtube.com/watch?v=eMlx5fFNoYc
related_formulas: [scaled-dot-product-attention, positional-encoding]
---

## The bottleneck attention was built to remove

A recurrent network, covered in the previous lesson, carries its entire view
of a sequence forward in one fixed-size hidden state, updated step by step.
For a long sequence, this becomes a genuine bottleneck. An enormous amount of
information about everything seen so far has to be compressed into a vector
of fixed size, and inevitably some of it gets lost — particularly
information from early in the sequence, by the time the network reaches the
end. **Attention** was introduced specifically to remove this bottleneck. At
each step, rather than relying solely on the compressed hidden state, the
model is allowed to look directly back at any earlier position in the
sequence and decide, dynamically, which of those earlier positions matter
most for producing the current output.

Take the sentence "The trophy didn't fit in the suitcase because it was too
big." Resolving what "it" refers to requires looking back at "trophy," not
the nearer word "suitcase" — a fixed-size hidden state has already blended
both words into one compressed summary by the time the model reaches "it,"
with no way to recover which one actually matters here. Attention sidesteps
that loss entirely: it lets the model weigh "trophy" more heavily than
"suitcase" directly, at the moment it is needed, rather than hoping the right
signal survived compression several steps earlier.

## The transformer: attention, and nothing sequential at all

A **transformer** takes this idea further than simply adding attention
alongside recurrence — it removes recurrence entirely, building the whole
architecture out of attention layers instead. Because there is no step that
depends on the step before it, every position in a sequence can be processed
simultaneously rather than one at a time. That is what makes transformers far
more parallelizable than an RNN, and therefore dramatically faster to train
on modern GPU hardware.

## Why this lesson stays brief on purpose

A full treatment of how attention actually computes which positions to
attend to — queries, keys and values, the scaling factor, multiple attention
heads, how position gets encoded since attention has no inherent sense of
sequence order — is a substantial topic in its own right. That is exactly
the territory Upshard's separate AI Engineer roadmap covers in depth, since
that mechanism is the foundation underneath every large language model
covered there. This lesson's job is narrower: establish why attention
mattered as an architectural idea, in contrast to the RNN approach this
roadmap just covered, rather than teach the full mechanics of how a
transformer is built. Readers who want the complete mechanical treatment will
find it there, including the formulas behind
[scaled dot-product attention](/formulas/scaled-dot-product-attention) and
[positional encoding](/formulas/positional-encoding).

## Where this leaves the sequence-modelling story

RNNs, with their sequential hidden state, and transformers, built entirely on
parallel attention, represent two genuinely different architectural answers
to the same underlying problem: how should a model handle data where order
matters? Transformers have displaced RNNs for most large-scale sequence
modelling today, particularly in language, largely because of the
parallelism advantage during training at the scale modern models operate at.
But the RNN's core idea — carrying a running summary forward through a
sequence — remains a useful, considerably simpler tool for smaller-scale
problems where the full architectural complexity isn't warranted.
