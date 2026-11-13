---
title: "Agents vs. workflows"
description: A fixed pipeline where code decides the next step is a workflow; a true agent is one where the model decides, including when to stop.
roadmap: ai-agents
node: agents-vs-workflows
group: foundations
video:
  provider: youtube
  id: "D7_ipDqhtwk"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 900
level: intermediate
prerequisites: [the-agent-loop]
takeaways:
  - A workflow is a fixed, predefined sequence of steps where code decides what happens next; an agent is a system where the model itself decides what happens next, including whether to stop.
  - Workflows trade flexibility for predictability — the same input reliably produces the same path through the system, which makes them cheap to run and straightforward to test.
  - Agents trade predictability for flexibility — they can handle situations nobody explicitly coded for, at the cost of being harder to test exhaustively and more expensive per run.
  - The two are not rivals with one correct winner; most production systems mix fixed workflow stages with an agentic stage where genuine judgment is required.
check_questions:
  - q: What is the core technical difference between a workflow and an agent?
    a: In a workflow, the developer's code determines the next step and when the process ends; in an agent, the model itself determines the next step and decides when it has finished, at run time.
  - q: Why are workflows generally cheaper and easier to test than agents?
    a: Because a workflow follows the same fixed sequence of steps for a given type of input, you can enumerate and test those paths directly; an agent can take a different number and order of steps each run, so its behavior space is much larger and harder to fully cover.
deeper:
  - label: "Agentic AI: Workflows vs. agents — Google Cloud Tech"
    url: https://www.youtube.com/watch?v=Qd6anWv0mv0
related_formulas: []
---

## Two ways to wire up the same loop

The agent loop — perceive, reason, act, observe — describes how a single agentic
step unfolds. But you can build the exact same application two structurally
different ways, and the difference comes down to one question: who decides what
happens after each step completes? If the answer is "a developer, in advance, in
the code," you have a workflow. If the answer is "the model, at run time, based on
what it just observed," you have an agent. Both can call the same tools and
produce similarly good final answers on an easy input. They diverge hard on
anything unexpected.

## What a workflow actually looks like

A workflow is a pipeline: step one always runs, its output always feeds into step
two, step two's output always feeds into step three, and so on, in an order fixed
at design time. An LLM call can be one of those steps — "summarize this
transcript," then "extract action items from the summary," then "format the
action items as an email" — but the *sequence* itself never changes based on what
the model says. If step two's output is garbage, the workflow still dutifully
hands it to step three. This sounds like a weakness, and it can be, but it is also
exactly what makes workflows valuable: given a fixed sequence of three LLM calls
with known inputs and outputs at each stage, you can write unit tests for each
stage independently, you can estimate cost up front because the number of model
calls per run never varies, and you can hand it to an operator who can trace
exactly where any output came from.

## What a true agent adds — and what it costs

An agent replaces the fixed arrows between steps with a decision the model makes
itself. Instead of "run step two next," the model is asked "given what you just
observed, what should happen next?" — and the honest set of answers includes
options nobody wrote into a pipeline, including the option to stop early or to
take an action twice. This is strictly more powerful for tasks where the right
sequence of steps genuinely cannot be known in advance. But that power is not
free. Because the model chooses its own path, two runs on inputs that look similar
can take a different number of steps, call different tools, or arrive at
differently-shaped answers — which makes an agent's behavior harder to test
exhaustively (you cannot enumerate every path the way you can for three fixed
pipeline stages), harder to budget for (a run that takes nine tool calls costs
roughly nine times one that takes one), and harder to debug after the fact, since
the "why" behind any given step lives in a model's reasoning rather than in a line
of code you can point to.

## A worked contrast

Consider "read this support ticket and draft a reply." This is naturally a
workflow: classify the ticket's category, pull the relevant policy text for that
category, and draft a reply using a fixed template — three steps, always in that
order, regardless of which ticket comes in. Wiring an agent loop around this task
would mostly add cost and unpredictability for no real benefit, because the right
sequence of steps is already known.

Now consider "investigate why this customer's account got flagged, and recommend
whether to reinstate it." Here, the right sequence of steps is not knowable in
advance: it might require checking login history, and if that looks fine,
checking payment disputes, and if a dispute shows up, pulling the related support
thread, and possibly a second payment check if the first one was ambiguous. A
fixed three-step pipeline cannot anticipate every branch this investigation might
need to take — a true agent can, because it decides its next move based on what
it actually finds at each step, the same way a human investigator would.

## The real lesson: name the right tool for the job

The practical skill this lesson is building toward is not "agents are better" or
"workflows are safer" — it is being able to look at a task and correctly judge
which shape it needs, because most production systems are neither purely one nor
the other. A support system might run a fixed workflow to classify and route a
ticket, then hand off to a genuinely agentic step only for the subset of tickets
that need real investigation. Knowing where that line sits, for a specific task in
front of you, is exactly the judgment the next lesson turns into a concrete test.
