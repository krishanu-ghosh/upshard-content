---
title: LangChain and LangGraph
description: "Why a chain-composition library grew a second, graph-based layer on top of it once agents needed to loop, branch, and remember state."
roadmap: ai-agents
node: langchain-and-langgraph
group: frameworks
video:
  provider: youtube
  id: "qAF1NjEVHhY"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 595
level: intermediate
prerequisites: [mcp-vs-custom-integrations, graph-and-dag-orchestration]
takeaways:
  - LangChain composes LLM calls into chains — prompt templates, output parsers, and tool wrappers linked together — which models an agent as a pipeline that runs mostly in one direction.
  - LangGraph models an agent as an explicit graph of nodes and edges sharing one persistent state object, which is what lets an agent loop, branch conditionally, and pause for a human without losing its place.
  - LangGraph exists as a separate layer, not a LangChain feature, because cycles and durable state break the chain abstraction's core assumption that execution flows forward through a fixed sequence of steps.
  - Reach for a chain when the steps are known in advance; reach for a graph when the agent itself decides which step runs next, and how many times.
check_questions:
  - q: Why can't a LangChain-style chain easily represent an agent that loops back and retries a step based on its own output?
    a: A chain is built as a mostly linear sequence of calls, so it has no native concept of returning to an earlier step — representing a loop means bolting conditional logic onto an abstraction designed for forward-only execution, which is awkward and fragile.
  - q: What does LangGraph's shared state object give an agent that a chain's step-to-step output passing does not?
    a: A single state object that every node can read and write means the agent's full context persists across branches, retries, and even pauses for human input, instead of being limited to whatever the immediately preceding step happened to output.
deeper:
  - label: "LangGraph Explained for Beginners — KodeKloud"
    url: https://www.youtube.com/watch?v=cUfLrn3TM3M
  - label: "Stop Confusing LangChain, LangGraph, and LangSmith | Full Breakdown — ByteMonk"
    url: https://www.youtube.com/watch?v=e-GR3PlEOVU
related_formulas: []
---

## A chain is a pipeline

LangChain's original idea is simple and genuinely useful: wrap the pieces around an
LLM call — a prompt template, the model call itself, a parser that turns the raw text
response into structured data, a retriever that pulls in documents, a tool wrapper
that lets the model invoke a function — into interchangeable components, then snap
them together into a chain. A chain for "summarize this document" might be
`load document -> chunk it -> prompt the model per chunk -> combine the summaries ->
format the output`. Each piece is swappable: change the prompt template, swap in a
different vector store for retrieval, change the model provider, and the rest of the
chain keeps working. That composability is the actual value LangChain still offers —
a large library of pre-built pieces for the unglamorous plumbing around a model call,
and a consistent interface for wiring them together.

## Where the chain abstraction runs out of road

A chain assumes execution flows forward through a sequence you mostly know ahead of
time. That assumption holds for "summarize a document" or "answer a question using
retrieved context," but it breaks the moment an agent needs to decide, at runtime,
whether to loop back and try again, which of several branches to take next, or to
pause mid-task and wait for a human to approve a step before continuing. You can
force a chain to do this with enough conditional glue code, but you are fighting the
abstraction rather than using it — the chain was built for a path, not a decision
tree the agent walks live.

## Why LangGraph became its own layer

LangGraph's answer is to stop modeling the agent as a sequence of calls and instead
model it as a graph: explicit nodes (each one a unit of work — call a tool, call the
model, check a condition), explicit edges connecting them, and one shared, persistent
state object that every node reads and can update as it runs. Crucially, edges can
point backward. A node that evaluates "did the tool call succeed" can route back to
a retry node, forward to the next step, or out to a human-approval node, all as
ordinary edges in the same graph. This is why LangGraph is a separate project built
on top of LangChain's primitives rather than a feature flag inside the chain API: a
graph with cycles and durable shared state is a different computational model from a
directed pipeline, not an extension of one. Trying to retrofit cycles into the chain
abstraction would have meant either breaking the simplicity that makes chains useful
for the linear case, or building a second system anyway — LangChain's maintainers
chose to build that second system explicitly instead of hiding it inside the first.

## A worked comparison

Picture a customer-support agent that looks up an order, and if the order is missing,
asks the customer a clarifying question before trying again. As a chain, this is
awkward: you'd need a conditional branch wedged into a structure built to assume
there is no going back. As a LangGraph graph, it's direct: a `lookup_order` node, a
conditional edge that checks whether the lookup succeeded, a `ask_customer` node it
routes to on failure, and an edge from `ask_customer` back to `lookup_order` once the
customer replies. The shared state object carries the order ID, the customer's
clarification, and the lookup result across every one of those hops, so by the time
the loop exits, every node downstream has the full history, not just whatever the
immediately prior node handed off.

## Choosing between them

The practical rule holds up well: if you can draw the task as a straight line of
steps that doesn't change shape based on what happens along the way, a LangChain
chain is simpler to write and reason about than a graph is. If the agent needs to
decide its own next step — retry, branch, wait for a person, revisit an earlier
stage — you need the graph model's cycles and shared state, and LangGraph is built
specifically for that. Many real systems use both: LangChain's component library
for the plumbing, assembled into nodes that LangGraph orchestrates. The split
reflects a real conceptual boundary — pipeline versus decision graph — not just two
competing products from the same company.
