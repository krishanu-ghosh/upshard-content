---
title: What is an AI engineer?
description: A discipline that builds products on models it did not train, sitting between the ML engineer who trains them and the developer who consumes an API.
roadmap: ai-engineer
node: what-is-an-ai-engineer
group: foundations
video:
  provider: youtube
  id: "NaKCw2p382s"
  start: 0
  end: null
creator:
  name: React Conferences by GitNation
  channel_url: https://www.youtube.com/@ReactConferences
duration_seconds: 1379
level: beginner
prerequisites: []
takeaways:
  - An AI engineer builds on foundation models rather than training them, which makes the job product engineering with a probabilistic component rather than research.
  - The discipline exists because capable models became available through an API, moving the scarce skill from training to everything that surrounds a model call.
  - Most of the work in a shipped LLM feature is retrieval, evaluation, cost and failure handling — the model call itself is the small part.
check_questions:
  - q: What separates an AI engineer from an ML engineer in practice?
    a: An ML engineer owns the model — data, training, weights. An AI engineer treats the model as a dependency and owns everything around it — context, tools, evaluation, cost and failure behaviour.
  - q: Why did this role appear only recently?
    a: Because capable models became reachable through an API. Once using a model no longer required training one, the scarce skill moved to the system built around it.
deeper:
  - label: "Thu Vu — Data Scientist vs. ML Engineer vs. AI Engineer (2026)"
    url: https://www.youtube.com/watch?v=vJeOpf5nhj0
related_formulas: []
---

## Why start with a job description

Because the rest of this roadmap only makes sense if you know which side of the
model you are standing on. Nothing here trains a network. Every lesson assumes
the weights already exist and somebody else paid for them, and asks what it
takes to build something trustworthy on top. That constraint is the whole
discipline. You cannot fix a model's mistake by training it away, so you fix
it with context, with retrieval, with a tool, with a check on the output, or
by choosing a different model. Those are the chapters that follow.

## The boundary with adjacent roles

A **data scientist** mostly answers questions about data that already
exists — what happened, why, what correlates with what — and ships notebooks
and dashboards more often than production services. An **ML engineer** owns
a model's lifecycle: curating training data, running training jobs, managing
experiment tracking, and deploying a model artifact fit on your own data for
your own task.

An **AI engineer** starts downstream of all of that. The model is a
dependency, versioned like any other, called over a network. The
engineering problem is building a reliable product around a component that's
probabilistic, occasionally wrong with total confidence, and that improves
or degrades on someone else's schedule whenever they ship a new version.

A useful rule of thumb: if the fix for a bug could be "collect more training
data and retrain," you are doing ML engineering. If the fix is "change the
prompt, retrieve a different document, add a tool, or validate the output
differently," you are doing AI engineering. Most teams building LLM features
today are entirely in the second category and never touch a training loop.

## What the job actually looks like week to week

In practice the work splits roughly into four buckets, and the split is the
most surprising thing to people arriving from traditional software
engineering.

A smaller slice than expected goes into the model call itself — writing the
prompt, choosing parameters — because that part is genuinely simple once
you've done it a few times. A much larger slice goes into **context
assembly**: deciding what the model should see, which means retrieval, tool
results, conversation history, and formatting, covered across this whole
roadmap. Another large slice goes into **evaluation and debugging**: reading
transcripts of what went wrong, building test cases out of real failures,
and measuring whether a change actually helped rather than just feeling
better. The last slice is **operational**: cost, latency, rate limits,
fallbacks, and the parts of running a service that have nothing to do with
AI specifically but are unavoidable once the feature is live.

This is also why the role rewards a different temperament than traditional
backend engineering. A REST endpoint either returns the right JSON or it has
a bug you can reproduce. An LLM feature can be "right" eighty-five percent of
the time by a measure you defined yourself, and the job is raising that
number steadily rather than reaching a provable one hundred.

## What has changed since this talk

The talk is three years old and the landscape it surveys has moved. At the
time, "AI engineering" mostly meant calling a chat completion endpoint with a
hand-written prompt. Since then, four things became their own disciplines
inside the role: **tool calling**, which lets a model act rather than only
answer; **context engineering**, the recognition that what goes into the
window matters more than how it is phrased; **agents**, systems where the
model chooses its own sequence of actions; and **evaluation as a practice**,
borrowed partly from machine learning and partly invented fresh, because
traditional software testing does not have a ready answer for "is this
answer good enough." Reasoning models — trained to produce extended internal
deliberation before answering — also arrived after this talk and changed how
much of the "thinking" work that used to live in a carefully engineered
prompt now happens inside the model itself.

The thesis aged better than the examples. If anything the gap it describes
has widened: the hard parts of shipping an LLM feature are now further from
the model call than they were, which is why this roadmap spends four groups
on retrieval, agents, evaluation and safety and only one on calling the model
at all.
