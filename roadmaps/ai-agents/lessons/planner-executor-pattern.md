---
title: "The planner–executor pattern"
description: Deciding the whole sequence of steps up front versus one step at a time, and why some designs do both.
roadmap: ai-agents
node: planner-executor-pattern
group: architectures
video:
  provider: youtube
  id: "uRya4zRrRx4"
  start: 0
  end: null
creator:
  name: LangChain
  channel_url: https://www.youtube.com/@LangChain
duration_seconds: 900
level: intermediate
prerequisites: [react-reason-and-act]
takeaways:
  - "The planner–executor pattern splits an agent into a planner, which decides the full sequence of steps up front, and an executor, which carries each step out in turn."
  - Planning ahead means fewer LLM calls and a clearer, inspectable structure than a step-by-step reasoning loop, because the whole route is fixed before execution begins.
  - That same upfront commitment is the pattern's weakness — a plan made before execution starts can go stale the moment an early step reveals something the planner didn't anticipate.
  - Replanning after each step, or after any step that surfaces new information, combines most of planning's efficiency with most of a reactive loop's adaptability.
check_questions:
  - q: "What is the core difference between the planner–executor pattern and a ReAct-style loop?"
    a: A ReAct-style loop decides one step at a time, reasoning fresh after every observation. The planner-executor pattern front-loads that reasoning into a single planning call that lays out the full sequence of steps before any of them run, and an executor then carries that fixed sequence out.
  - q: "Why can a plan go stale during execution, and what is the usual fix?"
    a: A plan is only as good as what was known when it was made, so if an early step returns something the planner didn't expect, the rest of the plan may no longer make sense. The usual fix is to replan after a step — regenerating the remaining plan in light of new information — rather than executing the original plan rigidly to the end.
deeper:
  - label: "Agent Design Patterns - Plan and Execute - Hands-on coding in LangGraph — AI Bites"
    url: https://www.youtube.com/watch?v=vpD9kf5Xwo0
related_formulas: []
---

## Two ways to structure an agent's steps

ReAct, covered earlier in this roadmap, interleaves reasoning and action one
step at a time: the agent thinks, acts, observes the result, and thinks
again, with every decision made fresh in light of what just happened. That
is one way to structure an agent's steps. The **planner–executor pattern**
is another: split the agent into two distinct roles. A **planner** — one LLM
call — looks at the task and produces the entire sequence of steps needed to
complete it, up front, before anything runs. An **executor** — a loop with
no planning responsibility of its own — then works through that sequence,
carrying out each step, possibly calling tools, until the plan is done.

## A worked plan

Take the task "Book a flight to Denver next Friday and email the
confirmation to my manager." A planner given this task, before touching any
tool, might produce something like:

1. Search flights to Denver for next Friday.
2. Select the cheapest option under $400.
3. Book the selected flight.
4. Retrieve the manager's email address from contacts.
5. Compose and send a confirmation email.

The executor then works through these five steps in order, calling the
flight-search tool for step one, a booking tool for step three, a contacts
lookup for step four, and an email tool for step five — none of which
requires the executor to decide *what* to do next, only *how* to carry out
the step it's already been handed. Compare this to a ReAct loop tackling the
same task: it would reason about step one, act, observe the flight results,
then reason fresh about what to do next, and so on five separate times, each
with its own full reasoning pass.

## The efficiency case for planning ahead

Because the entire sequence is decided in a single planning call, the
planner–executor pattern typically needs far fewer LLM calls for reasoning —
one planning call instead of one reasoning call per step — and the resulting
plan is a clear, inspectable artifact. You can read the five-step list above
and know exactly what the agent intends to do before it does any of it,
which makes the whole run easier to log, debug, and even show to a user for
approval before execution starts. A ReAct loop, by contrast, only reveals its
intentions one step at a time, as it goes.

## Where a fixed plan breaks

The same upfront commitment that makes planning efficient is also its
weakness. Suppose step one above returns no flights under $400 at all — only
options above $600. The plan's step two, "select the cheapest option under
$400," is now impossible to satisfy as written, because the planner made
that assumption without knowing real prices. A rigid executor that simply
marches through the fixed plan has no good way to handle this: the plan was
built on information that turned out to be wrong, and nothing in the
plan itself tells the executor what to do when that happens.

## Replanning: getting both properties

The common fix is **replanning**: after a step executes, instead of blindly
continuing to the next item on the original list, the system calls the
planner again, handing it the new information and asking it to produce an
updated remaining plan. In the flight example, after seeing that nothing is
under $400, a replan might change step two to "select the cheapest option
regardless of budget and flag the price difference to the user," then
continue from there. This costs an extra LLM call wherever a replan happens,
so it isn't free — but it only pays that cost when something actually
changed, rather than on every single step the way a full ReAct loop does,
landing the design between the two extremes: close to the efficiency of a
fixed plan, with most of the adaptability of reasoning fresh at every step.

## Choosing between the two

Neither approach is strictly better. A ReAct loop suits tasks where each
step's outcome is genuinely unpredictable and the next action depends
heavily on it — open-ended research, for instance. The planner–executor
pattern, especially with replanning, suits tasks that are structured enough
to lay out in advance but long enough that paying for fresh reasoning at
every single step would be wasteful. Recognizing which shape a task has is
itself part of designing the agent well.
