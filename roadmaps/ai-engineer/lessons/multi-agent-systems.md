---
title: Multi-agent systems
description: Orchestrators and sub-agents, handoffs, isolated context — and the honest case that one good agent usually beats five.
roadmap: ai-engineer
node: multi-agent-systems
group: agents
video:
  provider: youtube
  id: "u6jJcIFDLE4"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 900
level: advanced
prerequisites: [the-agent-loop]
takeaways:
  - The real argument for sub-agents is context isolation — a sub-task explores in its own window and returns a conclusion, not its mess.
  - Every handoff is a lossy serialization of state, and errors compound across a chain instead of canceling out.
  - Adding agents multiplies cost, latency and failure modes, and often produces worse results than one agent with better tools.
check_questions:
  - q: What is the strongest technical reason to split work across agents?
    a: Context isolation. A sub-agent can burn a whole window exploring and hand back only its conclusion, keeping the orchestrator's context clean for the decisions that matter.
  - q: Why do errors compound in a multi-agent chain?
    a: Each agent conditions on what the previous one passed along, and that's a lossy summary. A small early misunderstanding gets inherited and elaborated on rather than corrected, because no later agent sees the original.
deeper:
  - label: "Sandipan Bhaumik — Multi-agent orchestration patterns that hold up"
    url: https://www.youtube.com/watch?v=2czYyrTzILg
  - label: "Adam Terlson — Multi-agent systems as finite state machines"
    url: https://www.youtube.com/watch?v=OD13PiXw60o
related_formulas: []
---

## The appeal, and the arithmetic against it

Splitting a problem across specialists feels obviously right — it's how human
teams work, so a researcher agent, a writer agent, a critic agent, and an
editor agent, each with its own prompt and tools, coordinated by an
orchestrator, sounds like good design on first encounter.

The arithmetic says otherwise. Each agent is its own model call with its own
context, so cost scales roughly with the number of agents. Each handoff
serializes state into text — information that was clear and structured
inside one agent's context has to get compressed into a summary the next
agent can parse, and detail is lost at every one of those boundaries.
Reliability compounds in the wrong direction too: five sequential steps, each
succeeding 95% of the time on its own, multiply out to roughly 77% success
end to end, with no mechanism for a later agent to notice and fix a
misunderstanding an earlier one already baked in.

## The common topologies, and where each fits

A **sequential pipeline** runs agents one after another, each consuming the
previous one's output. Good for a task that genuinely decomposes into
ordered stages — draft, then critique, then revise.

A **parallel fan-out, fan-in** dispatches related sub-tasks to several
agents at once, then combines their independent results. Good for work
that's genuinely independent, like several unrelated research queries that
don't depend on each other's findings.

A **hierarchical orchestrator** has one agent keep overall control, deciding
which sub-agent to call for which piece and integrating the results itself,
rather than letting sub-agents hand off to each other directly. This is
usually the easiest topology to reason about and debug — there's always one
place to look for the state of the whole task.

A **peer-to-peer or debate** arrangement has agents challenge each other's
output directly, with no central coordinator. Useful when catching a
specific kind of error benefits from an adversarial framing rather than a
cooperative one — at the cost of being considerably harder to bound and
terminate cleanly.

## Context isolation: the strongest real argument

The most defensible reason to use more than one agent isn't specialization of
skill. It's isolation of context. A sub-agent investigating something can
spend its entire window exploring dead ends and reading irrelevant material,
then return only a few sentences of conclusion to whoever called it. The
orchestrator never sees the exploration — its own context stays clean and
focused on the decisions that actually matter, instead of being diluted by
the sub-agent's working process. This is the "isolate" move from
[context engineering](/roadmaps/ai-engineer/context-engineering), applied
across agents instead of within one.

## A worked cost comparison

Say a single well-equipped agent completes a task in about ten model calls.
Split the same task across a four-agent pipeline, where each agent needs
roughly three calls for its own narrower piece plus overhead for handoff
messages, and you can easily total twenty or more calls for the same work —
call it double the cost. You've also added three extra points where
information can get lost or distorted in translation, and three extra places
a failure can occur that then has to be traced back through the whole chain,
instead of through one agent's own trace.

## Before reaching for more agents

Try one agent with better tools first. A striking number of multi-agent
designs turn out to be compensating for a tool returning far too much raw
data, a single prompt trying to cover too many cases at once, or a context
that was simply never compacted as it grew. All of these are cheaper to fix
directly — and fixing them often removes the reason a second agent seemed
necessary in the first place.

## When splitting is still the right call

Genuinely parallel, independent work — several unrelated searches, several
files to examine at once — gains real wall-clock time from splitting, when
the branches truly don't depend on each other's results. Different tools or
permission levels is another legitimate reason: an agent permitted to write
to production should be a structurally different agent from the one reading
untrusted web content, with an explicit, inspectable boundary between them.

When you do split, make every handoff an explicit, structured format passed
between agents, not a loose paragraph of prose. And trace the entire
multi-agent run as a single connected unit — a chain of agents with no
end-to-end trace spanning all of them isn't meaningfully debuggable at all.
