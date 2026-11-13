---
title: Tracing and observability
description: Logging every prompt, tool call, and intermediate step an agent takes, so a human can actually tell why it did what it did.
roadmap: ai-agents
node: tracing-and-observability
group: evaluation-safety-and-deployment
video:
  provider: youtube
  id: "obfPzbapMIM"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 3240
level: intermediate
prerequisites: [evaluating-agent-performance, error-handling-for-tool-calls]
takeaways:
  - Tracing captures every model call, tool call, and intermediate input/output an agent produces, turning an opaque run into a structured record you can actually inspect.
  - A single bad final answer can originate from a bad plan, a bad tool call, a bad tool result, or a bad synthesis step, and without a trace there's no way to tell which one actually went wrong.
  - Tools like LangSmith and Langfuse exist specifically to capture, store, and visualize these traces for debugging in development and monitoring in production.
  - Traces that include sensitive data need masking before storage or display, since a debugging tool that logs everything can itself become a data-handling liability.
check_questions:
  - q: Why doesn't knowing an agent's final answer was wrong tell you what to fix?
    a: A wrong final answer could come from the agent planning the wrong approach, calling the wrong tool, misreading a correct tool result, or synthesizing a bad answer from correct evidence — four very different bugs that all produce the same symptom, and only a trace that records each step separately lets you see which one actually happened.
  - q: Why do tracing tools need to support masking sensitive data?
    a: A full trace captures every prompt and tool input/output verbatim, which often includes customer data, credentials, or other sensitive content passed through the agent, so the trace itself needs redaction before it's stored or shown to a developer, or the debugging tool becomes a new place that data leaks from.
deeper: []
related_formulas: []
---

## Why agents need tracing that single prompts don't

A single-turn LLM call has one input and one output — if it's wrong, you look
at the prompt and the response and you're mostly done. An agent run might
involve a planning step, three tool calls, two retries after a failed call,
and a final synthesis step that turns all of that into a user-facing answer.
If the final answer is wrong, "look at the input and the output" tells you
nothing, because the input and output you can see are several steps removed
from whichever step actually introduced the problem. Tracing exists to make
every one of those intermediate steps visible and inspectable after the fact,
the same way a stack trace makes a crash's call chain visible instead of
leaving you staring at the error message alone.

## What a trace actually records

A complete trace for an agent run captures, at minimum: the system prompt and
user input that started the run; every model call made during the run, with
its full prompt (including whatever the agent assembled from memory, retrieved
context, or prior tool results) and its full response; every tool call, with
the exact arguments passed in and the exact result returned; timestamps and
latency for each step, so you can see where time was spent; and any retries or
errors, with what triggered them. Tools like LangSmith and Langfuse structure
this into a tree or timeline view — a run is the root, and each model call or
tool call is a labeled span underneath it, nested in the order they happened.
This turns a run that might otherwise be an unreadable wall of concatenated
text into something you can expand, collapse, and step through span by span.

## A worked example: finding where a failure happened

Suppose a research agent is asked "what was Company X's revenue last quarter?"
and answers with a number that turns out to be from two quarters ago. Without
a trace, you have the question and the wrong answer, and no way to know why.
With a trace, you open the run and step through it in order:

1. **Planning step** — the agent decided to call a `search_filings` tool with
   the query "Company X quarterly revenue." Correct plan.
2. **Tool call** — `search_filings` returned three documents: the most recent
   quarterly filing, the one before it, and an annual report. Correct result;
   the right document was in there.
3. **Synthesis step** — the model's response quoted a revenue figure from the
   *second* document in the list, not the most recent one.

Reading the trace in order isolates the bug immediately: the retrieval step
worked fine, but the synthesis step picked the wrong document out of several
correct candidates — likely because nothing in the prompt told the model
which document was most recent, or the documents weren't ordered or labeled
by date. That is a specific, fixable problem (add explicit dates to each
retrieved document, or sort them before passing them to the model). Without
the trace, the same failure looks identical to a dozen other possible causes —
bad retrieval, a stale tool, a hallucinated number — and you'd be guessing at
a fix instead of applying one directly to the step that actually broke.

## From debugging tool to production monitor

Tracing isn't only for catching bugs during development. The same captured
traces, aggregated across thousands of production runs, become the basis for
monitoring: tracking average latency and cost per run, flagging runs where a
tool call failed or an agent looped past some step count, and feeding the
trajectories used by the LLM-as-judge evaluation covered in the previous
lesson. A production agent without tracing is a system you can only observe
by its outputs; one with tracing is a system whose internal decisions are
available whenever something needs explaining — to an engineer debugging a
regression, or to a judge model scoring whether a run behaved well.

## The cost of getting masking wrong

Because traces record full prompts and tool outputs verbatim, they often
contain exactly the sensitive data an agent was processing — a customer's
email thread, an account number, an internal document. A tracing setup that
stores all of this unmasked turns a debugging tool into a second place that
data is exposed, often with weaker access controls than the production
systems the data came from. Masking fields known to be sensitive — or running
output through a redaction step before it's written to the trace store — needs
to be part of the tracing setup from the start, not an afterthought added once
traces are already being collected and shared.
