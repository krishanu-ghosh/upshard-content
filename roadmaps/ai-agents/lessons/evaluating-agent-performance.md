---
title: Evaluating agent performance
description: Judging an agent means judging the whole trajectory it took, not just whether the final answer happened to be correct.
roadmap: ai-agents
node: evaluating-agent-performance
group: evaluation-safety-and-deployment
video:
  provider: youtube
  id: "spvXj9tnWAQ"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1500
level: intermediate
prerequisites: [langchain-and-langgraph, self-critique-and-reflection]
takeaways:
  - Evaluating an agent requires judging the full trajectory — the sequence of reasoning steps and tool calls it took — not just whether the final answer matches an expected output.
  - An agent can reach a correct final answer through a wasteful, unsafe, or simply lucky path, so final-answer-only grading systematically misses real failures.
  - LLM-as-judge evaluation, where a separate model reviews the entire trace against a rubric, scales human-style judgment across trajectories in a way exact-match scoring cannot.
  - Task-success-rate benchmarks run across many trials matter because a single run tells you almost nothing about an agent whose behavior varies run to run.
check_questions:
  - q: Why can an agent that gives the right final answer still have failed evaluation?
    a: The final answer only tells you the destination, not the path — an agent can arrive at a correct answer by calling the wrong tools, taking unsafe or wasteful detours, or guessing, and a trajectory like that is a failure even though the grader checking only the last message would mark it a pass.
  - q: Why do agent evaluations need to run many trials instead of one?
    a: Agents are often non-deterministic across runs because of sampling and the number of intermediate decisions involved, so one successful run does not establish that the agent is reliable; a task-success rate computed across dozens or hundreds of trials gives a real estimate of how often it actually works.
deeper:
  - label: "The Future of Evals: From LLM as a Judge to Agent as a Judge — Aparna Dhinakaran, Arize AI — AI Engineer"
    url: https://www.youtube.com/watch?v=q2JrUKBMf0w
related_formulas: []
---

## Why the final answer is the wrong unit of evaluation

Evaluating a single-turn LLM response is comparatively simple: you have a prompt,
an output, and some way to score that output against a reference. An agent breaks
this setup because its output is not one response — it is a sequence of decisions:
which tool to call, what arguments to pass, how to interpret the result, whether
to retry or change course, and finally what to report back. Scoring only the last
of those steps throws away everything that happened before it, which is exactly
where most agent failures actually live.

## A worked example: two agents, one right answer

Imagine two separate runs of the same customer-support agent, both asked "what's
the status of order #4521?" and both correctly answering "it shipped yesterday and
arrives Thursday."

**Trajectory A** calls `lookup_order(id=4521)`, reads the shipment status directly
from the result, and reports it. One tool call, directly relevant, answer grounded
in real data.

**Trajectory B** calls `list_all_orders()` (an expensive, unscoped call that returns
every customer's order history), scans the output for something that looks like
order 4521, misreads a different order's shipping date, calls `lookup_order(id=4521)`
as a second attempt after noticing the mismatch, and this time reports the correct
date. Both trajectories end at the identical final answer. A grader that only checks
the final message scores them as the same, identically successful run. In practice,
trajectory B leaked unrelated customers' order data into context it didn't need,
burned several times the tokens and latency, and only self-corrected because the
mismatch happened to be noticeable — a different query might not have triggered the
retry at all, producing a wrong answer with full confidence. Trajectory-level
evaluation is what tells these two runs apart; final-answer evaluation cannot.

## LLM-as-judge: reviewing the whole trace

Because trajectories are long, structured, and varied, you generally can't grade
them with exact-match rules the way you might grade a multiple-choice answer. The
common approach is LLM-as-judge: a separate model call is given the full trace —
every tool call, every tool result, every intermediate reasoning step, and the
final answer — along with a rubric describing what a good trajectory looks like
for this task. The judge model then scores the trajectory against that rubric:
did it use the minimum necessary tool calls, did it avoid unscoped or risky
actions, did it handle an error gracefully if a tool call failed, did the final
answer actually follow from the evidence gathered. This mirrors how a human
reviewer would read the same trace and reach a verdict, except it can be run on
thousands of trajectories where a human reviewer cannot.

A rubric matters more than the judge model itself. A vague instruction like
"did the agent do a good job?" produces inconsistent, superficial scores. A
rubric that spells out specific trajectory properties — number of tool calls
relative to the minimum needed, whether any call exposed data outside the
task's scope, whether failed calls were retried sensibly rather than silently
ignored — produces judgments that catch the kind of problem trajectory B
illustrates above.

## Task-success rate across many runs

A single successful trajectory, however well it scores, is still one data
point. Agents are frequently non-deterministic: sampling temperature, tool
results that vary slightly, and the sheer number of decision points mean the
same task run twenty times can succeed sixteen times and fail four different
ways. A task-success-rate benchmark runs the same (or a varied) set of tasks
many times and reports the fraction that complete successfully by whatever
trajectory-and-outcome standard you've defined. An agent that passes 95% of
500 trials is a meaningfully different, more trustworthy system than one that
happened to pass the one trial someone watched — and the gap between those two
numbers is invisible to anyone who only ever runs an agent once and reads the
transcript.

## Putting it together

Good agent evaluation combines both ideas: use an LLM judge (or a rubric-based
human review for high-stakes cases) to score the quality of each trajectory,
not just its ending, and run enough trials to turn individual trajectory scores
into a success rate you can actually trust and compare across versions of the
agent. Neither piece alone is sufficient — a single beautifully-judged
trajectory says nothing about reliability, and a success rate computed from
final-answer checks alone will systematically overrate agents that get lucky
the way trajectory B did.
