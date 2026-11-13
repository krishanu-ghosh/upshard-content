---
title: Tool calling
description: Declaring tools, executing what the model asks for, feeding results back — and designing tools a model can actually use correctly.
roadmap: ai-engineer
node: tool-calling
group: agents
video:
  provider: youtube
  id: "zuMw0pkPXpU"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1517
level: intermediate
prerequisites: [what-is-an-agent, structured-output]
takeaways:
  - The model never executes anything; it emits a structured request to call a tool, and your code decides whether and how to run it.
  - A tool definition is a prompt — its name, description and parameter docs are what the model reads to decide when to use it.
  - Errors should be returned to the model as content it can read and recover from, not raised as exceptions that end the turn.
check_questions:
  - q: What actually happens when a model calls a tool?
    a: It emits a structured object naming a tool and its arguments, and stops. Your code validates it, decides whether to run it, executes it, and appends the result to the conversation for the next turn.
  - q: Why return a tool error to the model rather than throwing?
    a: Because the model can often recover — fix an argument, try a different tool, or report the failure usefully. A thrown exception discards the run and loses the context that made recovery possible.
deeper:
  - label: "Philipp Schmid — Why senior engineers struggle to build AI agents"
    url: https://www.youtube.com/watch?v=3_gYbhABcAE
  - label: "Ilan Bigio (OpenAI) — Function calling, full workshop"
    url: https://www.youtube.com/watch?v=KUEmEb71vzQ
related_formulas: []
---

## The loop in miniature

You send the model a question along with a list of available tools, each
described by a name, a natural-language purpose, and a parameter schema.
Instead of answering directly, the model may emit a structured request —
this tool, with these specific arguments — built with the same constrained
decoding mechanism covered in
[structured output](/roadmaps/ai-engineer/structured-output). Your code then
validates that request, decides whether to actually run it, executes it, and
appends the result back into the conversation as a new message. The model
sees that result and either calls another tool or produces a final answer.

The single most important thing to internalise: the model has no execution
capability of its own. It produces a request — a proposal — and nothing
more. Whether that request is safe, permitted, rate-limited, or even
syntactically sensible is entirely your code's responsibility. Treating a
tool call as pre-authorised just because the model emitted it is the root
cause of most real agent security incidents.

## The mechanics of a multi-tool turn

A single model turn can request several tools in parallel — all independent
and safe to run concurrently — rather than one at a time, which matters for
latency when, say, three unrelated lookups are all needed before the next
step can proceed. Most APIs also expose an explicit **tool choice** setting:
`auto` lets the model decide whether to call a tool at all; `required` forces
it to call some tool rather than answer directly; a specific tool name forces
exactly that one; and `none` disables tool use for that call entirely, useful
for a final summarisation step where no further action should be taken.

## Designing a tool a model can actually use

**The description is the prompt.** The model chooses which tool to call based
on the description alone, nothing more. A function named `search` with the
docstring "searches" will be called essentially at random whenever anything
vaguely search-shaped comes up. A description reading "searches the customer
support knowledge base for articles about billing and account issues; returns
up to five article excerpts; use this for policy questions, not for looking
up the status of a specific order" will be called correctly, because it tells
the model both what the tool does and, just as importantly, what it is not
for.

**Fewer tools, each distinct.** Thirty tools with overlapping purposes
produce measurably worse selection accuracy than eight tools that are clearly
different from one another. If two tools are frequently confused in practice,
they usually want to be merged into one tool with a parameter distinguishing
the cases, rather than kept as two similar-sounding options.

**Make the arguments hard to get wrong.** Prefer an enumerated set of allowed
values over a free-text string wherever the real options are known in
advance. Put units explicitly in a parameter's name rather than leaving them
implicit. Mark a field required only when it genuinely always needs a value.
Everything validated by the schema itself is something that will never need a
retry to fix.

**Return errors as readable text, not a stack trace.** "No customer found
with id 4471 — ids are eight digits; did you mean an order id instead?" gives
the model a real chance to correct itself and try again sensibly. A raw
exception or an opaque error code does not, and typically just causes the
model to retry the exact same mistaken call.

**Keep results small and relevant.** A tool that returns a fifty-thousand-token
dump of raw data spends context budget that the rest of the task actually
needs. Return only what the next step genuinely requires, with a reference or
identifier the model can use to fetch more detail only if it turns out to be
necessary.

## Error handling as a first-class design concern

A tool call can fail for reasons that have nothing to do with the model: a
downstream service times out, a dependency is temporarily unavailable, an
external rate limit is hit. The agent loop needs its own retry policy at the
tool-call level, separate from the model-call retry policy covered in
[calling a model API](/roadmaps/ai-engineer/calling-a-model-api) — typically
a small number of attempts with backoff before surfacing the failure back to
the model as a readable error, so the model can decide whether to try a
different approach rather than looping on an identical failing call
indefinitely.
