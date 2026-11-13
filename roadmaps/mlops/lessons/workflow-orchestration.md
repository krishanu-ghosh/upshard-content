---
title: Workflow orchestration
description: Why multi-step ML pipelines need a scheduler that understands dependencies, retries, and failure — not a cron job and a prayer.
roadmap: mlops
node: workflow-orchestration
group: orchestration-infra
video:
  provider: youtube
  id: "CGxxVj13sOs"
  start: 0
  end: null
creator:
  name: Data with Marc
  channel_url: https://www.youtube.com/@MarcLamberti
duration_seconds: 710
level: intermediate
prerequisites: [continuous-delivery-for-models]
takeaways:
  - A workflow orchestrator represents a pipeline as a DAG of tasks with explicit dependencies, so the system — not a human — decides what can run, what must wait, and what to retry.
  - Idempotent tasks and automatic retries are what make a pipeline resilient to the routine failures (a flaky API, a slow upstream job) that would otherwise require someone to notice and manually re-run steps.
  - Orchestration tools separate "what should happen" (the DAG definition) from "did it happen" (the scheduler's run history), which is what makes a pipeline auditable and debuggable months later.
check_questions:
  - q: Why is a DAG the right data structure for representing an ML pipeline?
    a: Because pipeline steps have real dependencies — feature extraction must finish before training, training before evaluation — and a DAG encodes exactly those ordering constraints without allowing cycles.
  - q: What goes wrong if a multi-step pipeline is built as one long script instead of a set of orchestrated tasks?
    a: A failure partway through forces a full rerun from the start (wasting time and compute), there is no per-step retry or visibility into which step failed, and nothing enforces that steps only run after their actual dependencies complete.
  - q: What does "idempotent" mean in the context of a pipeline task, and why does the orchestrator depend on it?
    a: A task is idempotent if running it twice with the same inputs produces the same result without side effects like duplicate rows or double-charged resources, which is what makes it safe for the orchestrator to retry automatically.
deeper:
  - label: "Intro to Kubeflow Pipelines — Google Cloud Tech"
    url: "https://www.youtube.com/watch?v=_AY8mmbR1o4"
related_formulas: []
---

## The problem that shows up after the first pipeline works

The first time you get an ML pipeline running — pull data, validate it,
extract features, train, evaluate, register the model — it's usually a
shell script or a notebook you run top to bottom. That works exactly once,
by hand, while you're watching. It stops working the moment the pipeline
needs to run on a schedule, needs to recover from a failure without you
being awake, or needs to be shared with a teammate who has to understand
what it does without reading every line. A script has no concept of "this
step depends on that one," "this step failed, retry it three times before
giving up," or "skip the steps that already succeeded and only rerun what
broke." Workflow orchestration tools (Airflow, Kubeflow Pipelines, Prefect,
Dagster) exist specifically to add those concepts as first-class features
instead of ad hoc code you'd otherwise have to write and maintain yourself.

## Pipelines as DAGs, not scripts

The core idea is to represent a pipeline as a directed acyclic graph (DAG):
each task is a node, and an edge from task A to task B means "B cannot start
until A finishes successfully." Feature extraction depends on data
validation; training depends on feature extraction; evaluation depends on
training; deployment depends on evaluation passing a threshold. Writing this
as a graph instead of a linear script buys you two things a script can't
give you. First, independent branches can run in parallel automatically —
if you're building two separate feature sets from the same validated data,
the orchestrator can run both extraction jobs at once because nothing says
they depend on each other. Second, the orchestrator can reason about partial
failure: if training fails, it knows evaluation and deployment should not
run, but it also knows data validation and feature extraction already
succeeded and don't need to be redone. A plain script has no way to express
either of those facts — it just executes statements in order.

## Retries, idempotency, and why "just rerun it" isn't free

Most pipeline failures in practice aren't bugs in your logic — they're
transient: an API rate limit, a network blip, a worker node getting
preempted. An orchestrator's scheduler watches each task's exit status and
can automatically retry a failed task a configured number of times, often
with backoff, before alerting a human. This is enormously valuable, but it
only works safely if the task is idempotent — running it again with the
same inputs produces the same result rather than, say, appending a second
copy of the same rows to a table or triggering a second billable training
job that silently doubles your cloud spend. Idempotency isn't something the
orchestrator gives you for free; it's a property you have to design each
task to have (write-then-rename instead of append, versioned outputs keyed
by run ID instead of overwritten-in-place state). The orchestrator's retry
logic is only as safe as the tasks it's retrying, which is a genuinely easy
thing to get wrong the first time you build a pipeline this way.

## Scheduling, backfills, and the audit trail you didn't know you needed

Beyond dependency management, orchestrators handle *when* things run:
nightly retraining, hourly feature refreshes, or triggering off an upstream
event like new data landing in a bucket. They also handle backfills — if you
change the DAG and need last month's runs recomputed under the new logic,
the scheduler can replay historical runs without you writing a separate
script for it. And because every run is tracked (which tasks ran, when,
with what inputs, with what result), you get an audit trail that answers
"why does this model behave differently from the one we shipped last
Tuesday?" months after the fact — something a script you ran manually from
your laptop simply cannot answer, because there's no record it ever ran at
all.

## Why this matters going forward

Orchestration is the layer that turns a working pipeline into a *production*
pipeline: one that survives failures, runs unattended, and leaves a trail
when something needs debugging. Every later stage of an ML system —
continuous training, scaling inference, monitoring for drift — assumes
there's a reliable, inspectable pipeline underneath it doing the actual
work. Skipping this layer doesn't make the complexity go away; it just moves
that complexity into fragile scripts and the on-call engineer's memory,
which is exactly the failure mode orchestration tools were built to
eliminate.
