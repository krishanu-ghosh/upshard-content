---
title: Embeddings
description: Dense vectors whose geometry encodes meaning, where they come from, and what they are and are not good for.
roadmap: ai-engineer
node: embeddings
group: foundations
video:
  provider: youtube
  id: "wgfSDrqYMJ4"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 518
level: beginner
prerequisites: [how-llms-work]
takeaways:
  - An embedding is a dense vector produced by a learned model, positioned so that texts with similar meaning sit close together.
  - Closeness is measured by an angle rather than a distance, which is why cosine similarity is the default comparison.
  - Embeddings capture similarity of meaning, not truth, recency or authority — two confidently opposite statements embed close together.
check_questions:
  - q: Why is cosine similarity preferred to Euclidean distance for comparing embeddings?
    a: Because direction carries the meaning and magnitude mostly carries length and frequency artefacts. Comparing angles ignores that and keeps scores comparable across texts of different sizes.
  - q: Why can a retrieval system return a passage that says the exact opposite of the query?
    a: An embedding encodes what a text is about, not what it claims. "The drug is safe" and "the drug is not safe" are about the same thing and land close together.
deeper:
  - label: "Computerphile — Vectoring Words (Word Embeddings)"
    url: https://www.youtube.com/watch?v=gQddtTdmG_8
  - label: "StatQuest — Cosine Similarity, Clearly Explained!!!"
    url: https://www.youtube.com/watch?v=e9U0QAFbfLI
related_formulas: [cosine-similarity, dot-product, euclidean-norm]
---

## Meaning as a position

Search by keyword only finds text that uses the same words as your query.
The core problem embeddings solve: how do you find text that means the same
thing but uses completely different words? The answer is to turn meaning
into geometry.

An embedding model takes text and returns a fixed-length list of numbers —
typically a few hundred to a few thousand dimensions — chosen so that
position in that space carries meaning. Texts about the same thing end up
pointing in the same direction, and that single property is what makes
search over meaning, instead of keywords, possible.

Nothing about this is magic. The model is trained so that pairs of texts
known to be related — a question and its matching answer, a sentence and
its paraphrase, two chunks from the same document — get pushed closer
together in the vector space, while unrelated pairs get pushed apart. This
is usually done with a **contrastive** training objective: the model sees a
text, a genuinely matching text, and several unrelated texts, and gets
adjusted so the match scores highest among them. Doing this over millions of
such triplets is what shapes the geometry into something useful.

## What the numbers actually look like

A modern embedding model might return `768`, `1536` or `3072` numbers per
piece of text, and no individual number means anything on its own. There's
no dimension that corresponds to "is about finance" or "is a question."
Meaning lives only in the relationship between whole vectors, which is why
embeddings always get compared to each other and never read directly. Most
APIs normalize the output to unit length, so every vector lies on the
surface of a high-dimensional sphere — this is what makes cosine similarity
and the raw dot product interchangeable in practice, a point the
[similarity search](/roadmaps/ai-engineer/similarity-search) lesson relies
on.

## Why closeness is measured by angle

Two vectors pointing in nearly the same direction count as similar
regardless of how long either one is, because length mostly reflects
incidental properties like text length or word frequency, not meaning. A
short, punchy sentence and a long, hedging paragraph that express the same
idea should score as similar, and they do, because the comparison discards
magnitude and keeps only direction.

This is also precisely why an embedding can be confidently, uselessly
wrong. "The drug is safe at this dose" and "the drug is not safe at this
dose" are about the same medication, the same dose, the same property — so
they point in almost the same direction and score as highly similar, even
though they assert opposite things. An embedding encodes topic and framing,
not truth value.

## Newer techniques worth knowing

Two developments are common enough now to mention, even though neither
appears in the video.

**Matryoshka embeddings** are trained so a prefix of the full vector — say,
the first 256 of 1536 numbers — is still a valid, useful embedding on its
own, just a less precise one. This lets an application store a short vector
for a fast initial pass over a huge corpus and only use the full vector
where it matters, trading a little quality for a large reduction in storage
and compute.

**Late-interaction** models, by contrast, keep a separate vector per token
instead of collapsing a whole passage into one vector, and compare query and
document token by token at search time — more expensive, often noticeably
more accurate on short, keyword-dense queries.

Neither approach changes the one-vector-per-document story in this lesson so
much as it refines it. Both are worth knowing by name before you assume "an
embedding" always means one fixed vector.

## What embeddings are not for

They're not a fact store, not a ranking of quality, and not a substitute for
exact matching. A product code, a date, or a person's name is better found
by the keyword half of a hybrid search — which is why
[similarity search](/roadmaps/ai-engineer/similarity-search) doesn't rely on
vectors alone. And because an embedding model is itself a fixed, versioned
artifact, re-embedding your whole corpus is required whenever you switch
models. Vectors from two different embedding models aren't comparable to
each other at all, even if they happen to share the same number of
dimensions.
