---
title: LlamaIndex for agents
description: "A retrieval-first framework grew agent features on top of its indexing strength, which shows in exactly which agents it's the strongest choice for."
roadmap: ai-agents
node: llamaindex-for-agents
group: frameworks
video:
  provider: youtube
  id: "MmiveeGxfX0"
  start: 0
  end: null
creator:
  name: LlamaIndex
  channel_url: https://www.youtube.com/@LlamaIndex
duration_seconds: 998
level: intermediate
prerequisites: [rag-agents, vector-memory-and-rag-for-agents]
takeaways:
  - LlamaIndex began as a data-indexing and retrieval framework for connecting an LLM to your own documents, and its agent features, such as AgentWorkflow, were built on top of that existing retrieval layer rather than designed agent-first.
  - That origin shows up as a real strength for agents whose core job is finding the right information inside a large private document or data collection, where the indexing and query layer underneath the agent is doing most of the real work.
  - A general-purpose orchestration framework like LangGraph has no particular opinion about how you store or retrieve documents; LlamaIndex has strong, mature opinions there, at the cost of being less of a natural fit for agents whose job isn't primarily retrieval.
  - The right framework choice tracks what the agent spends most of its time doing, not which one has more features in the abstract.
check_questions:
  - q: Why does LlamaIndex tend to be a strong choice specifically for agents built around a large private document collection?
    a: LlamaIndex's agent layer sits on top of an indexing and retrieval system that was the framework's original focus, so the hard parts of chunking, embedding, and querying large document collections are already mature and well-integrated with the agent, rather than being left to the developer to wire up separately.
  - q: What's the practical tradeoff of choosing a retrieval-first framework for an agent that barely does any retrieval?
    a: You inherit a framework whose deepest strengths — document indexing, chunking strategies, query engines — go largely unused, while a more general-purpose orchestration framework might offer a simpler, more direct fit for an agent whose work is mostly tool calls and control flow rather than searching documents.
deeper:
  - label: "Building AI Agents that actually automate Knowledge Work - Jerry Liu, LlamaIndex — AI Engineer"
    url: https://www.youtube.com/watch?v=jVGCulhBRZI
  - label: "LlamaIndex Crash Course - AI Agents in Python — NeuralNine"
    url: https://www.youtube.com/watch?v=dkisb6FlU-g
related_formulas: []
---

## A framework built from the data outward

Most agent frameworks start from the agent loop and treat data access as one tool
among many. LlamaIndex started from the opposite end. Its first, and still central,
job is turning a pile of unstructured documents — PDFs, wikis, codebases, support
tickets — into an index a query can search efficiently, then handing the retrieved
chunks to an LLM to answer a question. The current agent layer is built on top of
that indexing and query machinery, not bolted onto a framework that was agent-first
from day one. That's a real architectural difference, not just branding: the
retrieval layer is the oldest, most developed part of the project, and its agent
abstractions are designed to call into that layer naturally.

Think of it like the difference between a search engine that grew a chat interface
and a chat app that grew a search feature. Both can answer a question with sources
attached, but the one built around search has years of unglamorous work underneath —
ranking, deduplication, query rewriting — that the other has to build from scratch.

## What this buys an agent that leans on retrieval

Consider an internal agent meant to answer employee questions by searching a
company's policy documents, past incident reports, and engineering wikis — tens of
thousands of pages, updated constantly, where the answer depends on finding the right
three paragraphs out of that pile. Building this well means solving problems that have
nothing to do with the agent loop itself. How do you chunk a hundred-page PDF so
relevant context doesn't get split across chunk boundaries? How do you combine results
from differently structured sources? How do you re-rank candidates before they hit the
model's context window, and keep the index current as documents change?

LlamaIndex has spent years accumulating answers to exactly these problems: multiple
chunking strategies, a range of index types beyond flat vector search, query engines
that route a question to the right sub-index. An agent built with LlamaIndex's
AgentWorkflow inherits all of that for free — a genuine head start over wiring
retrieval together by hand inside a more general-purpose orchestration framework.

## Where that strength stops mattering

Now consider a different agent: one that mostly calls a handful of internal APIs,
checks their results, and decides what to do next, with little or no document
retrieval involved. Here, LlamaIndex's deepest investment — its indexing and query
layer — sits mostly unused. The agent still needs what every framework has to
provide: a loop, state, tool-calling conventions, maybe branching logic. A framework
built agent-first, with orchestration as its central concern rather than a later
addition, is often the more natural fit, because its core abstractions were designed
around exactly that problem rather than grown out to cover it.

## Retrieval-first versus orchestration-first

This is the real axis to compare frameworks on. LangGraph and similar frameworks are
orchestration-first: a clear, general opinion about control flow — nodes, edges,
shared state — and little opinion about how you store or retrieve data, leaving that
to you or a library you plug in. LlamaIndex is retrieval-first: a clear, mature
opinion about indexing and querying data, with agent orchestration as the newer part
of the stack built to make use of that strength. Neither is a strictly better design.
They're optimized for different centers of gravity in what the agent actually spends
its time doing.

## Choosing with this in mind

A reasonable rule: if your agent's job mostly means "search across a large collection
of our own documents and answer based on what's found," LlamaIndex's retrieval-first
design is doing real, load-bearing work for you, and its agent layer is a sensible
default. If the job is mostly "call these tools, check results, decide what's next,"
with retrieval a minor piece if present at all, an orchestration-first framework
tends to feel more direct — you won't be carrying the weight of a retrieval system
you aren't really using. Many real systems combine both: a LlamaIndex-built retrieval
tool called from an agent loop built in something else, which is itself evidence that
these two strengths are complementary, not competing claims to the same job.
