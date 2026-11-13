---
title: The agent loop
description: "Perceive, reason, act, observe: the repeating cycle that lets an agent take several steps toward a goal instead of one."
roadmap: ai-agents
node: the-agent-loop
group: foundations
video:
  provider: youtube
  id: "AryOxmm9TGI"
  start: 0
  end: null
creator:
  name: "AI TL;DR"
  channel_url: https://www.youtube.com/@EngineeringTLDR
duration_seconds: 480
level: intermediate
prerequisites: [what-are-ai-agents]
takeaways:
  - The agent loop is perceive, reason, act, observe, repeated until the model decides the task is done or a stopping condition (a step limit, a budget, an error) is hit.
  - Each pass through the loop feeds the previous action's result back into the model's context, which is what lets the next decision be informed rather than blind.
  - The loop is structurally the same as a while-loop in ordinary code, except the condition that ends it — "do I have enough to answer?" — is evaluated by the model itself, not by a programmer's if-statement.
  - Multi-step tasks fail most often not because the model reasons badly in any one step, but because a step's observation is not fed back clearly enough for the next reasoning step to use it.
check_questions:
  - q: What are the four stages of the agent loop, in order?
    a: Perceive the current state and request, reason about what to do next, act by taking that step (often calling a tool), and observe the result, which feeds back into the next perceive stage.
  - q: How is the agent loop like a while-loop in ordinary programming?
    a: Both repeat a body of steps until a condition is satisfied; the difference is that a while-loop's condition is a fixed expression a programmer wrote, while the agent loop's condition — "is the task done?" — is judged by the model itself at each pass.
deeper:
  - label: "Perceive, Reason, Act Explained | The Core Loop Behind Every Agentic AI System — NeuralCanvas"
    url: https://www.youtube.com/watch?v=dLXmpkFmpBA
related_formulas: []
---

## The loop, stage by stage

The previous lesson established that an agent decides its own next action instead
of stopping after one answer. The agent loop is the concrete mechanism that makes
that possible, and it has four stages that repeat. **Perceive**: the model is
handed the current state — the original request, plus anything learned so far.
**Reason**: the model decides, given everything it currently knows, what the
single next step should be — answer now, or take an action first. **Act**: if the
model chose to act, that action actually runs — a tool call, a search, a
calculation — outside the model itself. **Observe**: the result of that action is
fed back into the model's context, becoming part of what it perceives on the next
pass. Then the cycle repeats from perceive, now with more information than it
started with, until the model's reasoning step concludes the task is finished.

## Why this looks exactly like a while-loop

In ordinary code, a `while` loop repeats a block of instructions as long as some
condition holds, and the loop body often updates a variable that the condition
checks. The agent loop has the same shape: the body is perceive-reason-act-observe,
and the condition being checked each time through is "does the model now have
enough information to produce a final answer?" The crucial difference is where
that condition lives. In a normal while-loop, a programmer wrote the condition in
advance — `while (results.length < 10)` — and the computer evaluates it
mechanically. In the agent loop, the condition is not written in code at all; it
is a judgment the model makes fresh at every pass, based on everything it has
observed. That is both the loop's power — it can handle situations nobody
anticipated — and its risk, since a model can misjudge "I'm done" just as easily
as it can misjudge any other reasoning step.

## A worked example across three iterations

Take the request: "Find the current weather in Tokyo and tell me if I need an
umbrella." Here is how the loop actually runs.

**Iteration 1.** Perceive: the agent reads the request and notices it has no
built-in knowledge of today's weather. Reason: it decides it needs to call a
weather tool before it can answer. Act: it calls `get_weather(city="Tokyo")`.
Observe: the tool returns something like `{"condition": "rain", "precip_chance":
80}`.

**Iteration 2.** Perceive: the agent now has the request plus that weather data in
its context. Reason: it checks whether this is enough to answer — yes, an 80%
chance of rain is a clear signal, so no further tool call is needed. Act: rather
than calling another tool, its "action" this pass is to compose a final answer.
Observe: there is nothing further to observe, because the action produced the
answer directly.

**Iteration 3 (or rather, the exit).** The loop's stopping condition is satisfied
— the model judged, on its own, that it had what it needed — so it returns: "Yes,
bring an umbrella — there's an 80% chance of rain in Tokyo today." Notice that a
genuinely harder version of this same request could have taken more iterations:
if the weather tool had returned an error, or an ambiguous multi-day forecast
instead of today's figure, the agent would reason again, perhaps retry the tool
with different parameters or call a second source, before reaching the same kind
of exit.

## Where loops actually break

The most common real-world failure in this loop is not the model reasoning
poorly — it is the observe step failing to hand back something the next reason
step can actually use. A tool that returns a huge, unfiltered JSON blob, or an
error message buried in boilerplate, can make a perfectly capable model reason
badly simply because the information it needed was present but not legible. This
is why later lessons on tool design and error handling matter as much as the loop
concept itself: the loop is only as good as what flows through it at each
observe stage.
