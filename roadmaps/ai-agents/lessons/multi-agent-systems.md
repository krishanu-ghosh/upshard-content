---
title: Multi-agent systems
description: Specialized agents collaborating on a task, and the real, debated tradeoff between that and one well-designed agent.
roadmap: ai-agents
node: multi-agent-systems
group: architectures
video:
  provider: youtube
  id: "2czYyrTzILg"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1560
level: intermediate
prerequisites: [planner-executor-pattern, self-critique-and-reflection]
takeaways:
  - A multi-agent system splits a task across several specialized agents — often a supervisor delegating to narrower worker agents — instead of one agent handling everything.
  - Specialization gives each worker a narrower prompt and a smaller toolset, which tends to make each individual agent's behavior more reliable and easier to reason about.
  - Multiple agents can also work in parallel on independent sub-tasks, something a single sequential agent cannot do on its own.
  - Coordination between agents is real overhead — more moving parts to debug, more places for miscommunication between agents — and a single well-designed agent genuinely outperforms a multi-agent setup on plenty of tasks, so this is a debated tradeoff, not an automatic win.
check_questions:
  - q: What does a supervisor agent typically do in a multi-agent system?
    a: A supervisor agent looks at the overall task, breaks it into sub-tasks, and delegates each one to a worker agent suited for it, rather than attempting the whole task itself. It coordinates the workers' outputs into a final result instead of doing the underlying work directly.
  - q: Why isn't splitting a task across multiple agents always the better choice?
    a: Multiple agents introduce coordination overhead — messages passed between agents, more places for a misunderstanding or dropped context to creep in, and more components to debug when something goes wrong. For many tasks, a single well-designed agent with a good prompt and tool set handles the whole thing more reliably and more cheaply than a multi-agent setup would.
deeper:
  - label: "Building Multi-agent Systems with Finite State Machines — AI Engineer"
    url: https://www.youtube.com/watch?v=OD13PiXw60o
  - label: "Why We Killed Our Multi-Agent Pipeline — Subbiah Sethuraman and Abhilash Asokan, ZS Associates — AI Engineer"
    url: https://www.youtube.com/watch?v=u6jJcIFDLE4
related_formulas: []
---

## Splitting the work instead of growing one agent

Every architecture covered so far in this roadmap — ReAct loops, the
planner–executor pattern, agentic RAG — describes how a single agent decides
what to do. A **multi-agent system** takes a different approach entirely: it
splits a task across several agents, each with its own prompt, its own
narrower toolset, and often its own specialty, rather than asking one agent
to hold all of that at once. The most common shape is a **supervisor**
agent that receives the overall task, breaks it into sub-tasks, and
delegates each one to a **worker** agent built for exactly that kind of
sub-task — then assembles the workers' results into a final answer.

## A worked example

Imagine a task: "Research our top three competitors' pricing pages and
write a one-page comparison." A supervisor agent could handle this with
three worker agents. A **research agent**, equipped only with web search and
page-fetching tools, is sent out three times, once per competitor, each run
returning a structured summary of that competitor's pricing. A **writing
agent**, with no tools at all beyond the model itself, takes the three
summaries and drafts the comparison page. The supervisor's own job is
narrow: decide which competitor to research next, pass each research
result to the writer, and hand back the finished draft. Each agent's prompt
stays focused — the research agent never needs to know how to write prose,
and the writing agent never needs to know how to drive a search tool —
which is a much simpler prompt to get right than one agent trying to do
both well at once.

## Why specialization helps

A single agent juggling research and writing needs a prompt that covers both
skills, a toolset that doesn't confuse the writing step with search tools,
and reasoning that correctly switches between "I am gathering facts" mode
and "I am composing prose" mode within one continuous run. Splitting these
into separate agents removes that switching entirely. Each agent's prompt
describes one job, which tends to produce more reliable, more predictable
behavior — and makes it far easier to tell, when something goes wrong, which
agent and which prompt is responsible. The three research runs in the example above
are also independent of each other — nothing about researching competitor A
depends on the result for competitor B — so a system built for it can run
those three sub-tasks in parallel, finishing in roughly the time one
research call takes rather than three.

## The coordination cost

None of this comes free. The supervisor now has to decide when each worker
has actually finished, what to do if a worker's result looks wrong or
incomplete, and how to merge results that might disagree or overlap — all
coordination logic that doesn't exist when there's only one agent making
all the decisions itself. Debugging gets harder too: a mistake could live
in the supervisor's delegation logic, in a worker's narrower reasoning, or
in how a worker's output got passed to the next stage, which means tracing
a bad outcome back to its cause now means checking several agents' behavior
instead of one. Every extra agent in the system is both an extra capability
and an extra place for something to break.

## A debated tradeoff, not an automatic win

It's tempting to treat "add more agents" as a reliable way to make a system
more capable, but that isn't a safe assumption. Teams building real
production systems have reported pulling multi-agent pipelines back out in
favor of one well-designed agent with a carefully written prompt and a
focused toolset, finding the single agent more reliable and far cheaper to
operate than the coordination overhead the multi-agent version demanded.
The right framing is that multi-agent systems trade simplicity for
specialization and parallelism — a trade that pays off when sub-tasks are
genuinely distinct and independent enough to benefit from it, as in the
competitor-research example, and that doesn't pay off when a task is simple
enough, or tightly coupled enough, that one agent handles it just fine on
its own. Deciding which situation you're in is the actual design skill
here, not reaching for multiple agents by default.
