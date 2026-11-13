---
title: When to use an agent
description: A practical test for whether a task's unpredictability is worth an agent's cost, latency, and occasional wrong turns.
roadmap: ai-agents
node: when-to-use-an-agent
group: foundations
video:
  provider: youtube
  id: "ePtYhwdRp2Q"
  start: 0
  end: null
creator:
  name: Apurv Khare
  channel_url: https://www.youtube.com/@ApurvKhare
duration_seconds: 600
level: intermediate
prerequisites: [agents-vs-workflows]
takeaways:
  - An agent is worth its cost only when the number or order of steps a task needs genuinely cannot be known in advance — if you can enumerate the steps, write a workflow instead.
  - A second real test is whether the task needs the model itself to judge when it is done, rather than finishing after a fixed number of predetermined steps.
  - Agents are also worth avoiding when you cannot tolerate occasional wrong turns — a task with a strict correctness bar or a hard latency budget usually favors a narrower, more predictable design.
  - Reaching for an agent by default, even for tasks a simple prompt or fixed pipeline would solve, adds cost, latency, and unpredictability with no corresponding benefit.
check_questions:
  - q: What is the first question to ask before deciding whether a task needs an agent?
    a: Whether the number and order of steps the task requires can be known ahead of time — if they can, a fixed workflow handles the task more cheaply and predictably than an agent would.
  - q: Why does "summarize this email and draft a reply" not need a real agent, while "research five competitors and compile a comparison" does?
    a: The email task has a fixed, known sequence of steps that never changes, so a workflow suffices; the research task requires an unknown number of searches whose targets depend on what earlier searches turn up, which only a system that can decide its own next step can handle well.
deeper:
  - label: "When to Build AI Agents vs AI Workflows? — Drew Patel"
    url: https://www.youtube.com/watch?v=b_tO2zxBq0M
related_formulas: []
---

## The decision is a cost-benefit question, not a philosophical one

Having a model that *can* decide its own next step does not mean every task
should let it. An agent costs more per run than a single prompt or a fixed
pipeline, because it typically makes several model calls instead of one, and each
of those calls adds latency the user waits through. It is also less predictable,
since the same input can take a different path on different runs. None of that is
a reason to avoid agents — it is a reason to reserve them for tasks where that
cost buys something a cheaper design cannot provide.

## A three-question test

Three questions, asked in order, cover most of the real judgment calls you will
face. **Is the number or order of steps genuinely unknown ahead of time?** If you
could sit down and write out the exact sequence of actions the task will need for
any reasonable input, you do not need an agent — a workflow will execute that
same sequence more cheaply and more predictably. **Does the task need the model to
decide when it's done?** Some tasks have a natural, fixed stopping point — "format
this text," "extract these three fields" — while others only reveal whether more
work is needed after you've already done some of it, which requires judgment a
fixed step-count cannot supply. **Can you tolerate occasional wrong turns?** An
agent that takes an unnecessary tool call, or occasionally searches the wrong
thing before correcting itself, is a normal cost of its flexibility — acceptable
for an internal research assistant, much less acceptable for a system issuing
refunds or sending customer-facing messages without review.

A task that answers "yes, yes, yes" to these three is a strong candidate for a
real agent. A task that answers "no" to the first one — because you can already
enumerate its steps — almost never needs one, regardless of how the other two
answers come out.

## Worked example: the email that doesn't need an agent

Take "summarize this email and draft a reply." Run the test: can you enumerate
the steps? Yes — read the email, produce a summary, produce a draft reply using
that summary and perhaps the sender's name and tone. That is two LLM calls in a
fixed order, every time, regardless of which email comes in. Does the model need
to decide when it's done? No — it is done after those two calls, full stop. Can
you tolerate wrong turns? This question barely matters here, because there is no
branching decision for the model to get wrong. Building this as an agent would
mean paying for a reasoning step that chooses "call the summarizer, then call the
drafter" — a choice that was never actually in doubt. A two-step workflow gets
the identical result for less money, less latency, and a design any engineer can
read top to bottom and know exactly what it will do.

## Worked example: the research task that does

Now take "research these five competitors and compile a comparison, using
whatever searches you find necessary." Can you enumerate the steps? No — one
competitor might have a clean public pricing page requiring a single search,
while another might require three separate searches to piece together pricing,
feature set, and recent news, and you cannot know which is which until you start
looking. Does the model need to decide when it's done? Yes — "done" here means
"I have enough reliable information about each competitor," a judgment that
depends on what the searches actually turn up, not on a step counter. Can you
tolerate wrong turns? Generally yes — an extra search that turns up nothing
useful just costs a little time and money; it does not corrupt the final
comparison, especially if a human reviews the output before it goes anywhere
important. All three answers point the same direction, which is exactly what
makes this a genuine agent task rather than a workflow wearing an agent's
marketing.

## The default should be the cheaper option

Given the choice, default to the workflow and only escalate to a real agent when
the test above actually fails for the workflow — specifically, when you cannot
write down the steps in advance. Teams that reach for an agent as a default,
rather than as a deliberate choice earned by genuine unpredictability, end up
paying agent-level cost and latency for tasks that a two-step pipeline would have
handled just as well, with a result that is easier to trust precisely because it
is easier to predict.
