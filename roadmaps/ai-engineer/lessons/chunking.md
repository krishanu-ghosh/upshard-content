---
title: Chunking
description: Splitting documents for retrieval, and why chunk boundaries decide retrieval quality more than the embedding model does.
roadmap: ai-engineer
node: chunking
group: retrieval
video:
  provider: youtube
  id: "ZTOtxiWb2bE"
  start: 0
  end: null
creator:
  name: Redis
  channel_url: https://www.youtube.com/@Redisinc
duration_seconds: 505
level: intermediate
prerequisites: [tokens-and-tokenization]
takeaways:
  - A chunk is the unit that gets embedded and retrieved, so a badly cut chunk cannot be rescued by a better embedding model or a better query.
  - Fixed-size splitting is fast and ignores meaning; recursive splitting respects document structure; semantic splitting cuts where the topic changes and costs more to build.
  - Overlap exists so that a passage straddling a boundary still appears whole somewhere, at the cost of duplication in the index.
check_questions:
  - q: Why does chunk size affect retrieval quality in both directions?
    a: Chunks that are too small lose the context that makes a passage meaningful, so they retrieve poorly. Chunks that are too large dilute the embedding across several topics, so the vector matches nothing sharply.
  - q: What problem does overlap solve, and what does it cost?
    a: It keeps a passage that crosses a boundary intact in at least one chunk. It costs index size and introduces near-duplicate results that then need deduplicating.
deeper:
  - label: "Greg Kamradt — The 5 levels of text splitting for retrieval"
    url: https://www.youtube.com/watch?v=8OJC21T2SL4
  - label: "Think in Models — When each chunking strategy fails"
    url: https://www.youtube.com/watch?v=9ztE_pX7Ltc
related_formulas: []
---

## The decision everything downstream inherits

Retrieval can only ever return a chunk you already created. If the sentence
that answers the question got split across two chunks, no embedding model,
no re-ranker, and no amount of prompt work will put it back together. That's
why chunking deserves more attention than it usually gets — it's the one
stage whose mistakes are unrecoverable later. It's also the stage most often
left at a library default. A thousand-character fixed window with two
hundred characters of overlap is a reasonable starting point and a poor
finishing point for most real corpora.

## The three strategies, worked through on one paragraph

Take a paragraph explaining a refund policy: one sentence stating the
policy, one giving the time limit, one listing exceptions.

**Fixed-size splitting** cuts at a character or token count regardless of
meaning. If the cut falls mid-sentence, between the time limit and the
exceptions, a query asking about exceptions retrieves a chunk that states
the limit but not the exceptions — and the answer comes back half-true.

**Recursive splitting** respects the document's own structure: cut at
paragraph breaks first, then sentence breaks, and only fall back to a raw
character count if nothing structural is nearby. The refund paragraph, being
one coherent block, is far more likely to survive intact, or at worst split
along a sentence boundary that still leaves each piece meaningful on its own.

**Semantic splitting** goes further, embedding individual sentences and
cutting wherever consecutive sentence similarity drops sharply — detecting a
topic change directly instead of inferring it from formatting. This earns
its keep on unstructured transcripts or scanned prose with no headings or
paragraph marks to exploit, at the cost of an embedding pass over the whole
corpus just to decide where to cut.

## Size and overlap, with concrete numbers

A chunk of roughly 256 to 512 tokens is a common, reasonable middle ground:
small enough that an embedding represents one coherent idea instead of
several blended together, large enough to keep the surrounding sentence or
two of context a single fact usually needs to stay interpretable on its own.
Overlap of ten to twenty percent of the chunk size — fifty to a hundred
tokens on a five-hundred-token chunk — keeps a sentence sitting near a
boundary intact in at least one chunk, even if it's split in its neighbor.
The cost is storing some duplicated text and later deduplicating
near-identical results that both legitimately matched.

## Chunking different content types

Plain prose chunks reasonably well by sentence and paragraph structure
alone. Code should be chunked along function or class boundaries rather than
by character count — splitting a function body mid-statement produces a
chunk that's syntactically meaningless on its own. Tables are often best
kept whole, or chunked row-group by row-group with the header repeated in
every chunk, since a table row without its column headers loses its meaning
entirely.

PDFs carry their own hazard: text extraction frequently loses the original
layout, so a two-column page can interleave unrelated sentences from each
column into one garbled stream before chunking ever begins. A multi-column
PDF is worth checking by eye at least once before trusting its extracted
text.

## An advanced pattern: small chunks, larger parents

**Parent document retrieval** splits a document into small chunks purely for
matching — small chunks embed more precisely, since they represent one
narrow idea — but when a small chunk is retrieved, the system hands the
model the larger surrounding section it came from, not the tiny matched
piece alone. That's the best of both sizes: precise matching from small
chunks, full surrounding context in what actually reaches the model.

## Measuring whether your chunking is any good

The only honest test is retrieval recall: take real questions, mark which
chunk should answer each one, and measure how often that chunk actually
appears in the retrieved set. A chunking strategy that looks reasonable on
inspection but scores poorly here is the one to revisit, before touching
anything downstream in the
[RAG pipeline](/roadmaps/ai-engineer/rag-pipeline).
