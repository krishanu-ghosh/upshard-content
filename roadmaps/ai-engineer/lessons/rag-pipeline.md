---
title: The RAG pipeline
description: Ingest, chunk, embed, index, retrieve, re-rank, generate — the whole path, with the failure mode waiting at each stage.
roadmap: ai-engineer
node: rag-pipeline
group: retrieval
video:
  provider: youtube
  id: "tLMViADvSNE"
  start: 0
  end: null
creator:
  name: Cole Medin
  channel_url: https://www.youtube.com/@ColeMedin
duration_seconds: 771
level: intermediate
prerequisites: [chunking, similarity-search, context-engineering]
takeaways:
  - A RAG system is two pipelines — an offline one that builds the index and an online one that answers a query — and they fail differently.
  - Most quality problems are retrieval problems wearing a generation costume, so measure what was retrieved before blaming the model.
  - Grounding is not automatic; without citations and an explicit instruction to abstain, a model will answer from parameters when retrieval returns nothing useful.
check_questions:
  - q: A RAG answer is wrong. What do you check first?
    a: Whether the correct passage was retrieved at all. If it was not, the problem is upstream in chunking, retrieval or re-ranking, and no prompt change will fix it.
  - q: Why does a RAG system need an explicit instruction to say it does not know?
    a: Because a model given a question and irrelevant context will still produce a fluent answer from its parameters. Abstention has to be an option you ask for, and an eval case you test.
deeper:
  - label: "Google Cloud Tech — Advanced RAG techniques for developers"
    url: https://www.youtube.com/watch?v=sGvXO7CVwc0
related_formulas: [cosine-similarity]
---

## Two pipelines, not one

A RAG system is really two separate pipelines wearing one name, and mixing
them up in your head is how debugging goes wrong. The offline pipeline runs
whenever documents change: load, clean, chunk, embed, index, store metadata —
think of it as stocking a library's shelves. The online pipeline runs per
query: embed the question, retrieve candidates, re-rank, assemble context,
generate, cite — that's the librarian answering a question using whatever is
already on the shelves. They fail at different times. The offline one fails
quietly at build time, and the failure doesn't surface until weeks later, as
poor answers that are hard to trace back to the shelf where things first went
wrong.

## The pipeline, stage by stage, with its characteristic failure

**Load and clean.** PDFs lose their original layout on extraction, tables
become scrambled word soup, running headers and footers repeat identically on
every single page and pollute every chunk drawn from that document. Garbage
introduced at this stage is invisible until much later and propagates into
every downstream step.

**Chunk.** Covered in its own
[lesson](/roadmaps/ai-engineer/chunking). The characteristic failure is a cut
running straight through the sentence that would have answered the question.

**Embed and index.** The failure here is a quiet mismatch — embedding the
corpus with one model and the live queries with a different one, or changing
embedding models without fully rebuilding the index from scratch. Either
produces a system that returns confident, plausible-looking, wrong results,
because the two sets of vectors are not comparable even though nothing throws
an error.

**Retrieve.** The failure is recall: if the chunk that actually answers the
question never makes it into the candidate set at all, nothing downstream —
no re-ranker, no clever prompt — can recover it. This is measured directly by
taking real questions, marking which chunk should answer each one, and
checking how often that chunk appears anywhere in, say, the top fifty
candidates before re-ranking.

**Re-rank.** The failure is skipping this stage entirely, trusting the
initial retrieval order as if it were already the final one — as
[similarity search](/roadmaps/ai-engineer/similarity-search) covers, this is
the cheapest large quality improvement available and the one most often left
out.

**Assemble.** The failure is over-stuffing the prompt with ten marginally
relevant documents that crowd out the two genuinely good ones, exactly the
dilution problem described in
[context engineering](/roadmaps/ai-engineer/context-engineering).

**Generate and cite.** The failure is ungrounded confidence: a model asked a
question with irrelevant or missing retrieved context will still produce a
fluent, specific-sounding answer drawn from its own parameters rather than
from anything actually retrieved.

## Query rewriting, and why the raw user question is often a poor query

A user's literal question is frequently a worse search query than a rewritten
version of it. Conversational context gets lost — "what about the second
one?" means nothing to a retriever with no memory of what "the first one" was
in a prior turn — and a question phrased as a question ("why does my request
time out after exactly thirty seconds?") often retrieves worse than a
rewritten, more document-like statement ("request timeout after thirty
seconds, cause"). A rewriting step, using the model itself to reformulate the
user's question into one or more better search queries before retrieval ever
runs, is a cheap and often substantial quality improvement, and it is also
where **multi-hop retrieval** becomes possible: a question requiring two
facts from two different documents can be split into two sequential
retrieval queries, with the result of the first informing the second, rather
than hoping a single retrieval pass happens to surface both at once.

## Grounding is not automatic, and has to be demanded

A model given a question and some retrieved context does not automatically
restrict itself to that context — left unconstrained, it will happily blend
retrieved facts with whatever it already knows from training, producing an
answer that is subtly wrong in ways that are hard to catch because most of it
is correct. Grounding requires an explicit instruction: answer only from the
provided context, cite which specific passage supports each claim, and state
plainly when the context does not contain enough to answer — plus an
unanswerable question deliberately included in the
[eval set](/roadmaps/ai-engineer/building-an-eval-set) to verify that
abstention actually happens rather than merely being requested.
