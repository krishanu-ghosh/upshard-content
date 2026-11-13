---
title: RAG agents
description: Retrieval as a decision the agent makes mid-task, not a fixed step that always runs once before generation.
roadmap: ai-agents
node: rag-agents
group: architectures
video:
  provider: youtube
  id: "ynJyIKwjonM"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 3780
level: intermediate
prerequisites: [vector-memory-and-rag-for-agents, what-are-tools]
takeaways:
  - "Agentic RAG treats retrieval as a tool call the agent decides to make, potentially more than once, rather than a fixed step that always runs exactly once before generation."
  - A single query can require multiple separate retrievals when it actually contains several sub-questions that live in different parts of a knowledge base.
  - The agent judges each retrieved result for sufficiency before answering, and issues a follow-up search instead of generating from incomplete context when the first pass falls short.
  - This flexibility costs extra latency and LLM calls compared to naive RAG, so it earns its keep on genuinely multi-step questions, not every lookup.
check_questions:
  - q: What is the key difference between naive RAG and agentic RAG?
    a: Naive RAG always retrieves exactly once, before generation, regardless of what the question actually needs. Agentic RAG lets the agent decide whether to retrieve at all, and whether to retrieve again, treating the search itself as a tool call made as part of its reasoning rather than a fixed pipeline step.
  - q: Why might a single question require two separate retrieval calls instead of one?
    a: A question can bundle two sub-questions whose answers live in different parts of a knowledge base, so one retrieval tuned for the first sub-question will not surface the passages the second one needs. An agentic RAG system searches once per sub-question and merges what comes back, rather than hoping one query covers both.
deeper:
  - label: "Building Production-Ready RAG Applications: Jerry Liu — AI Engineer"
    url: https://www.youtube.com/watch?v=TRjq7t2Ms5I
related_formulas: []
---

## From a fixed step to a decision

The retrieval-augmented generation you met earlier in this roadmap, when an
agent reaches into a vector store for relevant memory, has a naive default
shape: embed the incoming query, pull back the top-k matching chunks, and
stuff them into the prompt before the model ever generates a word. That shape
works fine for a single, self-contained question against a single source of
truth. It breaks down the moment a question needs more than one lookup, or
no lookup at all, because naive RAG has no mechanism to notice either case —
it retrieves exactly once, unconditionally, every time.

**Agentic RAG** removes that unconditionality. Instead of retrieval being a
pipeline stage that always fires, it becomes a **tool** sitting alongside
whatever other tools the agent has, callable whenever the agent's own
reasoning decides it needs more information. The agent can skip it entirely
for a question it can already answer, call it once, or call it several
times in sequence — each call informed by what the previous one returned.

## Why one retrieval often is not enough

Consider a question like "What's the refund window for annual plans, and
does it differ for customers who upgraded mid-year?" This is really two
sub-questions stitched together. A naive RAG system embeds the whole
sentence as one query and searches once, which tends to surface passages
about refund windows in general while missing anything specific to the
mid-year-upgrade edge case — the two topics simply don't share enough
semantic weight in a single embedding to both surface strongly.

An agentic RAG system handles this differently. It first retrieves for the
refund-window sub-question, reads back a clear answer — say, thirty days —
and then, rather than generating immediately, notices the second
sub-question hasn't been addressed yet. It issues a **second, separate**
retrieval specifically about mid-year upgrades, pulls back a passage noting
that upgraded accounts get a prorated fifteen-day window instead, and only
then composes a final answer that correctly distinguishes the two cases.
Naive RAG, with its single fixed retrieval, would have had to get lucky that
both facts appeared in the same top-k chunk pull — and usually wouldn't.

## Judging sufficiency, not just fetching

The mechanism underneath this is a judgment the agent makes after each
retrieval: is what came back enough to answer, or not? This is the same
reasoning move covered elsewhere in this roadmap under self-critique — the
agent evaluates its own intermediate state rather than blindly proceeding.
Concretely, after the first search in the example above, the agent's
reasoning trace looks something like: "I have the refund window. I still
don't know about mid-year upgrades. I should search again, more
specifically, for that." That explicit check — do I have what I need, or do
I need to search again — is the part naive RAG skips entirely, because it
never looks at what it retrieved before deciding whether to retrieve more.

## What this costs

None of this is free. Each additional retrieval is a round trip to the
vector store, and each decision about whether to retrieve again typically
costs an extra LLM call to reason about sufficiency. A question naive RAG
answers in one retrieval plus one generation, agentic RAG might answer in
two retrievals plus two or three LLM calls — several times the latency and
token cost for the same final answer. For a simple, single-fact lookup, that
overhead buys nothing, which is exactly why well-designed agents don't
always reach for the agentic version: the decision to retrieve adaptively is
itself a design choice, made for questions complex enough to need it, not a
strict upgrade applied everywhere retrieval appears.

## Where this fits in an agent's toolset

Framed this way, agentic RAG is not a separate architecture competing with
the agent loop you've already learned — it's naive RAG's retrieval step
promoted to a first-class tool, reasoned about the same way the agent
reasons about calling a calculator or a web search. The judgment of when to
retrieve, and whether the first retrieval was enough, is just the agent
applying its general reasoning loop to one specific tool rather than
treating that tool as a mandatory, unconditional first move.
