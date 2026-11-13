---
title: What is an agent?
description: A model using tools in a loop to reach a goal — where that genuinely helps, and where a fixed pipeline is the better engineering.
roadmap: ai-engineer
node: what-is-an-agent
group: agents
video:
  provider: youtube
  id: "D7_ipDqhtwk"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 909
level: intermediate
prerequisites: [chain-of-thought]
takeaways:
  - An agent is a model that chooses its own next action from a set of tools, in a loop, until a goal is met — the model decides the control flow.
  - A workflow with model calls at fixed steps is not an agent, and for a known task it is cheaper, faster and far easier to debug.
  - Agents earn their cost when the path to the answer genuinely cannot be known in advance, and lose it everywhere else.
check_questions:
  - q: What distinguishes an agent from a pipeline that calls a model several times?
    a: Who decides what happens next. In a pipeline the sequence is code you wrote. In an agent the model picks the next action each iteration, so the control flow is generated at runtime.
  - q: Why is an agent harder to operate than a workflow?
    a: Non-determinism. The same input can take a different path each run, so cost and latency vary, failures do not reproduce, and you cannot test the path — only the outcome.
deeper:
  - label: "Chip Huyen — Why people think agent is a buzzword, but it isn't"
    url: https://www.youtube.com/watch?v=D6v5rlqUIc8
  - label: "Sam Bhagwat — Agents vs workflows, why not both"
    url: https://www.youtube.com/watch?v=8SUJEqQNClw
related_formulas: []
---

## The definition worth keeping

An agent is a model using tools in a loop to pursue a goal. The load-bearing
word is *loop*: the model acts, sees the result, and decides what to do next,
with nobody specifying in advance how many steps that takes or what order
they come in. That one property is the whole trade. You get a system that can
handle tasks whose shape you could not predict in advance. You also get a
system whose control flow is generated at runtime by a probabilistic process,
which is exactly as hard to operate as that sounds.

## A spectrum, not a binary

It helps to think of a range of designs rather than a strict line between
"agent" and "not agent."

At one end sits a fixed **workflow**: every step and its order are written in
code, and a model call might appear at one or two points to classify or
extract, but the model never chooses what happens next. One step up, a
**router** uses a model call purely to decide which of several fixed
downstream paths to take — still no open-ended looping, just a model-made
branch. Further along, an **orchestrator-worker** pattern has one model plan
a sequence of sub-tasks and dispatch each to a worker, with the orchestrator
keeping control over the overall sequence even though workers may operate
somewhat independently. At the far end sits a fully autonomous agent: the
model decides its own next action, evaluates the result, and decides again,
for as many iterations as the task demands, bounded only by the stopping
conditions covered in [the agent loop](/roadmaps/ai-engineer/the-agent-loop).

Most production systems that call themselves agents actually sit somewhere
in the middle of this spectrum — one genuinely open-ended loop embedded
inside an otherwise fixed pipeline.

## What distinguishes a workflow from a loop, concretely

Say you write a pipeline as a function: classify, then retrieve, then
generate, then validate, each step calling the next. That sequence is code
you authored and can read in full before ever running it. In an agent, the
model itself decides, at runtime, which tool to call next based on what the
previous tool returned — and that decision isn't visible until the run
actually happens.

The practical consequence is testing. A workflow's path through the code can
be unit tested step by step, because the path is fixed. An agent's path
can't be tested the same way: the same input can legitimately take a
different route through the tools on two separate runs. That's why agents
are evaluated on whether they reach a correct final outcome, covered in
[evaluation](/roadmaps/ai-engineer/why-evals-matter), rather than on whether
they followed a specific predicted sequence of steps.

## Why an agent is operationally harder

Non-determinism is the root of the difficulty. The same input can take a
different number of steps on different runs, so cost and latency vary
noticeably run to run rather than being predictable in advance. A failure in
production often doesn't reproduce on a second attempt, because the random
element in sampling can send the agent down a different path the second
time. And debugging a bad outcome means reconstructing an entire multi-step
trace after the fact — covered in
[tracing and observability](/roadmaps/ai-engineer/tracing-and-observability)
— rather than reading a stack trace that points directly at the failing
line of code.

## When a workflow wins, and when an agent earns its cost

If you can draw the steps on a whiteboard before writing any code, write them
as a workflow. A pipeline that classifies, retrieves, generates and validates
in a fixed order is testable at every stage, costs a predictable amount per
run, fails in reproducible and diagnosable ways, and completes in a fixed
amount of time. The same work done by an open-ended agent is slower, more
expensive on average, and occasionally inventive in ways nobody asked for.

The deciding questions: can the steps genuinely be enumerated in advance?
Does the correct next step depend on information that only becomes available
partway through — information a fixed sequence couldn't have anticipated?
And is a wrong turn along the way recoverable rather than catastrophic?
Enumerable steps with no real dependency on intermediate discovery mean write
the workflow. A genuinely unpredictable path, where recoverable mistakes are
an acceptable cost of flexibility, is where the open-ended loop earns its
complexity.
