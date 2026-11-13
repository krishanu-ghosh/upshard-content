---
title: Deploying and scaling agents
description: What changes when an agent has to run as a real service under concurrent load, instead of as a script on one machine.
roadmap: ai-agents
node: deploying-and-scaling-agents
group: evaluation-safety-and-deployment
video:
  provider: youtube
  id: "HT4l0DeP69I"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1200
level: intermediate
prerequisites: [tracing-and-observability, crewai-and-autogen]
takeaways:
  - Running an agent in production means handling many concurrent requests reliably, not just getting one request to work end to end on a laptop.
  - Deploying via containers, managed agent platforms, or infrastructure-as-code replaces the long-running local script with something that can be restarted, scaled, and updated without someone babysitting a terminal.
  - Prompt caching cuts cost by reusing the computation for a prompt's unchanged prefix — the system prompt, tool definitions, retrieved context — across repeated calls instead of reprocessing it every time.
  - Routing cheaper sub-tasks to smaller or cheaper models, reserving the most capable model for the steps that actually need it, is a direct lever on cost at scale.
check_questions:
  - q: What specifically breaks when a local agent script is exposed to real concurrent traffic instead of being run one request at a time?
    a: A script written to handle one request at a time typically assumes it owns shared state like conversation history or a single API connection, so concurrent requests can corrupt each other's state, exhaust rate limits meant for one caller, and have no mechanism to queue, retry, or isolate failures — problems that simply don't appear when you're the only one calling it.
  - q: How does prompt caching reduce the cost of running an agent at scale?
    a: Many of an agent's calls share a large unchanged prefix — the system prompt, tool schemas, and often retrieved context — and prompt caching lets the model provider reuse the already-processed representation of that prefix across calls instead of reprocessing every token from scratch each time, which cuts both cost and latency for the repeated portion.
deeper:
  - label: "How to Actually Tokenmaxx and Save Millions — Alvin Pane, OutRival — AI Engineer"
    url: https://www.youtube.com/watch?v=8g1N_a3h9H0
related_formulas: []
---

## From a working script to a running service

An agent that works when you run it from the command line on your own machine
has cleared a low bar: one request, one user, no competition for resources,
and a failure just means you read the stack trace and try again. Production
deployment asks a completely different question — can this same agent serve
many users at once, recover from a failed step without a human watching,
keep its cost per request from running away, and be updated without taking
the whole thing down? None of that is implied by "it works," and all of it is
usually invisible until the first time real traffic hits the thing.

## A worked example: what actually changes

Picture a local script: it loads a system prompt, enters a loop that calls an
LLM, executes whatever tool call comes back, feeds the result back in, and
exits when the agent produces a final answer. It stores conversation state in
a plain Python list in memory. It calls external APIs directly and prints
errors to the console if something fails.

Turning this into a service that can actually be deployed changes nearly
every one of those assumptions:

- **Concurrency.** The in-memory list keyed to one running process has to
  become per-request or per-session state, usually in an external store (a
  database, a cache), because multiple requests are now running through the
  same deployed instance at once and can't share one list without corrupting
  each other's conversations.
- **Retries and failure handling.** A tool call that fails midway can no
  longer just print an error — it needs a retry policy (with backoff, and a
  cap so a persistently failing call doesn't loop forever) and a defined
  fallback behavior, because nobody is watching the console to notice and
  restart it by hand.
- **Observability.** As covered in the tracing lesson, a service running
  unattended needs its traces and metrics captured automatically, since a
  production failure with no trace is a failure nobody can diagnose.
- **A deployment pipeline.** Instead of running `python agent.py` on a
  laptop, the agent gets packaged — typically as a container — and deployed
  through a managed agent platform or infrastructure-as-code (so the exact
  environment, dependencies, and configuration are reproducible), with health
  checks that restart it automatically if it crashes and a path to roll out
  an updated version without downtime.

None of these changes touch the agent's actual reasoning or tool logic — the
part that was "the agent" to begin with. They're entirely about the
difference between a script that runs once when you launch it and a service
that has to keep running correctly under conditions nobody is watching in
real time.

## Cost control: prompt caching

Every agent call typically resends a large, mostly-unchanging prefix: the
system prompt describing the agent's role, the full set of tool schemas it
can call, and often a block of retrieved context. Without caching, the model
reprocesses every token of that prefix on every single call, even though it's
often identical call to call within a session or even across sessions.
Prompt caching lets the provider store the already-computed representation of
that unchanged prefix and reuse it on subsequent calls, billing the cached
portion at a steep discount (and serving it faster) compared to processing it
fresh. For an agent that might send the same several-thousand-token system
prompt and tool schema set on every one of a dozen steps in a single run, this
is often the single largest cost lever available — a cost reduction that
costs nothing in terms of changing the agent's actual behavior.

## Cost control: routing sub-tasks to cheaper models

Not every step in an agent's loop needs the most capable model available.
Classifying which of five tools to call, summarizing a short tool result, or
extracting a field from structured output are all tasks a smaller, cheaper
model handles about as well as the frontier model the agent might use for its
hardest reasoning step. Model routing assigns each sub-task to the cheapest
model that can do it reliably, reserving the expensive model for steps that
actually need its capability — the complex planning step, the final synthesis
where mistakes are costly. An agent that routes its five easy sub-tasks to a
model costing a fraction as much per token, while keeping its one hard
reasoning step on the frontier model, can cut its total per-run cost
substantially without any measurable drop in the quality of its final output,
simply because most of an agent's steps were never the ones that needed the
expensive model in the first place.
