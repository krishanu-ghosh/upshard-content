---
title: Similarity search
description: Scoring, top-k, hybrid search with BM25, and re-ranking — why vector search alone underperforms on real queries.
roadmap: ai-engineer
node: similarity-search
group: retrieval
video:
  provider: youtube
  id: "r0Dciuq0knU"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 508
level: intermediate
prerequisites: [vector-databases]
takeaways:
  - Dense vector search finds meaning and misses exact strings; sparse keyword search does the opposite, which is why hybrid retrieval beats either alone.
  - Fusing two ranked lists is usually done on rank rather than score, because scores from different retrievers are not on a comparable scale.
  - A re-ranker scores query and document together and is far more accurate than either retriever, which is why it runs last over a short candidate list.
check_questions:
  - q: Why does vector search fail on a part number or an error code?
    a: An embedding encodes meaning, and an identifier has almost none — two different codes look nearly identical in vector space. Keyword search matches the exact string, which is what the query needs.
  - q: Why combine two retrievers by rank instead of by score?
    a: A cosine score and a BM25 score are on unrelated scales with no shared calibration, so adding them weights one arbitrarily. Rank is comparable across both.
deeper:
  - label: "Dave Ebbelaar — Hybrid search measured stage by stage on a real corpus"
    url: https://www.youtube.com/watch?v=XvKiTfd6Xvo
related_formulas: [cosine-similarity, dot-product]
---

## Two retrievers with opposite weaknesses

Picture two different librarians. One understands what you mean even when
you phrase it badly; the other has memorized the exact spelling of every word
in every book but no sense of meaning at all. Real retrieval needs both.

Dense retrieval embeds the query and finds vectors pointing the same way. It
handles paraphrase and synonym effortlessly — a query about "cancelling a
subscription" matches a document about "ending a recurring plan" even though
they share almost no words. But it's nearly useless for an exact token: a
SKU, a version string, a surname, an error code carries little semantic
meaning to embed, so two different codes can look nearly identical in vector
space. Sparse retrieval, in practice **BM25**, scores documents on term
overlap with the query, weighted by how rare each term is across the whole
corpus. A word appearing in every document contributes almost nothing to the
score; a word appearing in only a few documents is a strong signal when it
matches. BM25 nails the identifier and misses the paraphrase entirely. Their
failure modes barely overlap, which is the whole argument for running both
and merging the results.

## Scoring, worked through numerically

Cosine similarity between two vectors is their dot product divided by the
product of their lengths — see
[dot product](/formulas/dot-product) and
[cosine similarity](/formulas/cosine-similarity) for the full definitions. A
toy two-dimensional illustration: a query vector of (1, 0) and a document
vector of (0.9, 0.1) have a dot product of 0.9, lengths close to 1, and a
cosine similarity near 0.9 — very similar. The same query against a document
vector of (0, 1) has a dot product of exactly 0, a cosine similarity of 0 —
completely dissimilar, the vectors are at right angles. When vectors are
normalised to unit length, which most embedding APIs do automatically, cosine
similarity and the plain dot product become the same number, which is why the
two terms are often used interchangeably in practice and why that is not
sloppy terminology so much as a genuine equivalence under normalisation.
Euclidean distance is also offered by most vector indexes, and on normalised
vectors it ranks results identically to cosine similarity — but mixing
metrics between how a corpus was indexed and how a query is scored produces
silently wrong rankings, so the choice has to be made once and kept
consistent.

BM25's score for a document, roughly, rewards a query term appearing more
often within that one document (term frequency), rewards a term that is rare
across the whole corpus more than one that is common (inverse document
frequency), and penalises a document for being unusually long relative to the
corpus average, since a long document matching a term by sheer volume of text
is a weaker signal than a short, focused one matching the same term.

## Fusing two ranked lists, worked through

**Reciprocal rank fusion** merges two independently ranked lists without
needing their raw scores to be on a comparable scale at all — which they
never are, since a cosine score and a BM25 score live on entirely unrelated
numeric ranges. Each document is scored as one divided by a small constant
plus its rank in a given list, and these per-list scores are summed across
every list the document appeared in. A worked example: document A is ranked
first by the dense retriever and fifth by BM25; document B is ranked third by
both. Using a small constant of 60, document A scores roughly 1/61 + 1/65 ≈
0.0318, while document B scores 1/63 + 1/63 ≈ 0.0317 — nearly tied, with
document A's strong dense rank narrowly outweighing its weaker keyword rank.
The method needs no tuning or calibration and is hard to beat for the effort
it costs to implement.

## Re-ranking: a different, slower, more accurate model

A **cross-encoder** reads the query and a single candidate document together,
as one combined input, and outputs a single relevance score for that pair —
in contrast to the dense retriever's **bi-encoder**, which embeds the query
and every document independently and never lets them interact before
comparison. Reading them jointly lets a cross-encoder make far finer
distinctions, but it means scoring one query against a thousand documents
requires a thousand separate model calls, which is far too slow to run over
an entire corpus. The standard architecture is therefore a funnel: a cheap
bi-encoder or hybrid search retrieves a broad set of maybe fifty candidates
quickly, and the expensive cross-encoder re-ranks only those fifty, handing
the top handful to the model. Most of the quality improvement in a mature
retrieval system comes from this final re-ranking stage, and it is the step
most often skipped by systems that stop at raw vector search.
