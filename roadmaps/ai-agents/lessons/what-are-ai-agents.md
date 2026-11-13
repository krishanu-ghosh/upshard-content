---
title: What are AI agents?
description: An AI agent perceives, decides, and acts in a loop toward a goal — a different shape from a model that only answers one prompt.
roadmap: ai-agents
node: what-are-ai-agents
group: foundations
video:
  provider: youtube
  id: "F8NKVhkZZWI"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 720
level: intermediate
prerequisites: []
takeaways:
  - An AI agent is a system built around an LLM that can perceive input, decide what to do, and take actions toward a goal, repeating that cycle until the goal is met.
  - "A plain chatbot call is stateless and single-shot: you send a prompt, it returns text, and the interaction is over regardless of whether the answer was actually useful."
  - An agent decides its own next step, which means it can call a tool, look at the result, and choose to call another tool or stop, all without a human re-prompting it in between.
  - The defining feature of "agentic" behavior is not intelligence but autonomy over the next action — the model, not your code, decides what happens next.
check_questions:
  - q: What makes a system an "agent" rather than a plain LLM call?
    a: An agent decides its own next action based on what it observes, and can take multiple actions in sequence toward a goal, rather than returning a single answer to a single prompt.
  - q: If you send ChatGPT one question and it replies with one answer, is that an agent?
    a: No — that is a single-shot, stateless call. It becomes agentic only if the system can act, observe a result, and decide to act again on its own before handing back a final answer.
deeper:
  - label: "AI Agents Explained in 14 Minutes (The Complete Guide) — Aishwarya Srinivasan"
    url: https://www.youtube.com/watch?v=TZMdEg1ZoIo
related_formulas: []
---

## A single prompt is not an agent

Ask a plain LLM "what's the weather in Tokyo right now?" and it will do its best
with what it already knows, which — for a fact that changes hourly — is nothing
useful. It cannot check anything. It generates a plausible-sounding response from
patterns in its training data, hands it back, and the interaction ends there. That
is the entire shape of a normal chatbot call: one input, one forward pass through
the model, one output, done. The model has no way to notice that its own answer
might be wrong, and no mechanism to do anything about it even if it did.

An AI agent is built to close exactly that gap. It wraps the same underlying model
in a loop that lets it look at the world, decide on an action, take that action, and
look again before answering. The model is still doing the same next-token
prediction it always does — nothing about the underlying LLM changes. What changes
is the scaffolding around it: code that gives the model tools to call, feeds their
results back in, and lets the model decide, turn by turn, whether it has enough to
answer or needs to act again.

## The three capabilities that define an agent

Strip away the hype and an agent needs exactly three things working together.
**Perception** is the ability to take in information beyond the original prompt —
a tool's output, a file's contents, a piece of retrieved context. **Reasoning** is
the step where the model looks at what it has and decides what to do next: answer
now, or gather more information first. **Action** is the ability to actually do
something in the world — call an API, run a search, write a file — rather than
only producing text that describes what it would do. A system is missing "agent"
status if any one of these is absent. A tool-calling function with no reasoning
loop around it is just a function. A model that reasons step by step in its own
output but never acts on anything is just a verbose chatbot.

## A worked example: booking logic versus agent logic

Suppose a user asks, "Is there a flight from Boston to Denver under $300 tomorrow,
and if so, hold it?" A single-shot model can only guess at an answer, because it
has no access to live fares. A scripted integration — a fixed pipeline wired by a
developer — could call one flight-search API, apply a price filter, and return a
result; this works, but only for exactly this query shape, and it cannot adapt if
the first airline's API is down or the user's phrasing implies a different
airport.

An agent handling the same request perceives the request, reasons that it needs a
flight-search tool, acts by calling it, and observes the results. If the first
search returns nothing under $300, the agent can reason again — try a nearby
airport, widen the time window, or call a second airline's API — before deciding
whether to hold a ticket or report back that nothing qualifies. Nobody wrote code
for "if the first search is empty, try a nearby airport." The model decided that
on its own, mid-task, based on what it observed. That capacity to improvise the
next step from an observed result, rather than following a path a developer
specified in advance, is the real dividing line this lesson is pointing at.

## Why the word "agent" gets overused

Not everything marketed as an agent earns the term. A chatbot with a slightly
longer system prompt is still a single-shot call. A "workflow" that always
executes the same five steps in the same order, regardless of what happens at step
two, is automation, not agency — later lessons in this group draw that line more
precisely. What makes the IBM Technology framing useful here is its insistence
that autonomy over the next step, not vocabulary or marketing, is the test: does
something other than a human or a fixed script decide what happens after the
first action? If yes, you are looking at an agent. If no, you are looking at a
pipeline with an LLM bolted on, which is a perfectly fine thing to build — it is
just not an agent, and the rest of this roadmap depends on being honest about
which one you actually have.
