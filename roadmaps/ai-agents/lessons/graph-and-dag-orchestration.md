---
title: Graph and DAG orchestration
description: Modeling an agent's workflow as explicit nodes and edges, with branching and shared state, instead of one implicit loop.
roadmap: ai-agents
node: graph-and-dag-orchestration
group: architectures
video:
  provider: youtube
  id: "GE7mqOhAj_o"
  start: 0
  end: null
creator:
  name: Gaurav Kumar Jain
  channel_url: https://www.youtube.com/@gauravkumarjain
duration_seconds: 1080
level: intermediate
prerequisites: [multi-agent-systems, planner-executor-pattern]
takeaways:
  - "Graph orchestration represents an agent's workflow explicitly as nodes (steps or agents) and edges (what happens next), instead of hiding that control flow inside a single implicit loop."
  - A conditional edge lets the graph branch to one of several next nodes based on the current state, which is how explicit if/else-style control flow gets built into an agent's workflow.
  - State is passed along the graph's edges and shared between nodes, so a later node can act on what an earlier node decided or produced without re-deriving it.
  - This explicitness is the LangGraph model's core idea, and it trades some of a simple linear chain's minimalism for control flow that's visible and inspectable rather than buried in code.
check_questions:
  - q: What does a conditional edge in a graph-based agent workflow actually do?
    a: A conditional edge inspects the current state after a node runs and decides which node to go to next based on that state, rather than always proceeding to the same fixed next step. This is how branching logic — effectively an if/else — gets expressed directly in the graph's structure instead of being buried inside a single node's code.
  - q: How does state move through a graph-based workflow, and why does that matter?
    a: State is passed along the graph's edges from node to node, so each node receives whatever earlier nodes have written to it and can both read and add to it. This matters because it lets a later node make decisions based on what happened earlier in the run without needing to recompute or re-fetch that information itself.
deeper:
  - label: "Architecting and Testing Controllable Agents: Lance Martin — AI Engineer"
    url: https://www.youtube.com/watch?v=ib-wTAvCZqg
related_formulas: []
---

## Making control flow explicit

Every architecture in this roadmap so far — a ReAct loop, a planner feeding
an executor, a supervisor delegating to workers — has a control flow that
lives implicitly in code: an if-statement here, a while-loop there, deciding
what runs next. **Graph and DAG orchestration** takes that control flow and
makes it a first-class, explicit structure: the workflow is represented
directly as a **graph**, with **nodes** standing in for steps or agents, and
**edges** standing in for "what happens after this node runs." Instead of
reading through code to figure out what an agent can do next, you can read
the graph itself. This is the model LangGraph, covered later in this
roadmap, is built around — a workflow is a graph you define, not a loop you
write by hand.

## Nodes, edges, and state

A node in this model is simply a unit of work — it could call an LLM, call
a tool, or run an agent — that reads the current **state** and returns an
update to it. An edge connects one node to the next. The simplest edges are
unconditional: node A always leads to node B. But the more powerful kind is
a **conditional edge**: after a node runs, a small function looks at the
resulting state and chooses which of several next nodes to go to, based on
what it finds. This is exactly how explicit branching — the graph
equivalent of an if/else statement — gets built into the workflow's
structure rather than hidden inside one node's internal logic.

**State** itself travels along these edges. Each node can read whatever
state the nodes before it produced, and add its own contribution before
passing it onward, so information doesn't need to be re-derived or
re-fetched at every step — a later node simply inherits what earlier nodes
already worked out.

## A worked example: router versus linear chain

Compare two graphs handling customer messages. A **linear chain** has three
nodes in a fixed sequence: classify the message, draft a response, send it
— A leads to B leads to C, always, no matter what the message says. This
works, but it treats every message identically regardless of what kind of
message it actually is.

A **router** node changes this. After classifying the message, instead of
always proceeding to the same next node, a conditional edge inspects the
classification stored in state and branches: if the state says
`category: billing`, it routes to a `billing-agent` node equipped with
refund and invoice tools; if the state says `category: technical`, it
routes instead to a `tech-support-agent` node equipped with diagnostic
tools. Both branches eventually converge on the same `send-response` node,
which reads whichever agent's output got written to state and sends it —
that node doesn't need to know which branch it came from, only that a
response is waiting in state for it to send.

The linear chain could not express this at all without burying an if/else
inside the single "draft a response" node, mixing billing logic and
technical-support logic into one undifferentiated block of code. The graph
version keeps them as separate, inspectable nodes, with the branching
decision itself visible as a distinct conditional edge rather than hidden
inside either node's implementation.

## Why explicitness is the whole point

The payoff of drawing a workflow as a graph rather than writing it as
nested conditionals is mostly about visibility. You can look at the graph
above and immediately see every path a message can take — through billing,
through tech support, or (if a third branch existed) through neither — and
exactly which node reads and writes which piece of state. Debugging a run
means walking the same graph: which node ran, what state looked like going
in and coming out, which edge fired next. None of that requires reading
through nested code to reconstruct what the control flow actually was,
because the control flow *is* the graph.

## The cost of that structure

A graph is more structure than a simple linear chain needs when a workflow
genuinely has no branching at all — defining nodes, edges, and a shared
state schema for three steps that always run in the same order is more
scaffolding than three function calls in sequence would require. The
graph model earns its cost once a workflow has real branches, loops back to
an earlier node, or multiple agents that need to share state cleanly — which
is exactly the kind of workflow multi-agent systems and planner–executor
designs tend to produce once they grow past their simplest form.
