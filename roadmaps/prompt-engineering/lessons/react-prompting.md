---
title: ReAct prompting
description: Interleave reasoning with real actions and their observations in a loop — the pattern behind most tool-using agents.
roadmap: prompt-engineering
node: react-prompting
group: advanced-patterns
video:
  provider: youtube
  id: "vFdIrZyKEwQ"
  start: 0
  end: null
creator:
  name: Akshay Pachaar
  channel_url: https://www.youtube.com/@akshay_pachaar
duration_seconds: 345
level: intermediate
prerequisites: [tree-of-thoughts]
takeaways:
  - ReAct interleaves explicit reasoning ("Thought"), real actions ("Action"), and their results ("Observation") in a repeating loop, rather than reasoning in one block and acting separately.
  - Grounding each reasoning step in a fresh observation from the real world catches wrong assumptions immediately, instead of letting them silently shape the rest of a plan.
  - The pattern is framework-agnostic — it describes a prompting structure, not a specific library, even though most tutorials demonstrate it through one.
check_questions:
  - q: What are the three repeating parts of a ReAct loop?
    a: A Thought, where the model reasons about what to do next; an Action, where it invokes a tool such as a search or calculator; and an Observation, where the result of that action is fed back in before the next Thought.
  - q: Why does interleaving actions with reasoning catch errors that pure reasoning misses?
    a: Because each Thought is grounded in a real Observation from the previous Action rather than an assumption the model made up, a wrong guess about the state of the world gets corrected by the next real result instead of silently propagating through the rest of the reasoning.
deeper:
  - label: "Sam Witteveen — Understanding ReACT with LangChain"
    url: https://www.youtube.com/watch?v=Eug2clsLtFs
related_formulas: []
---

## Reasoning that touches the real world

Every technique covered so far in this group — step-back, self-consistency,
tree of thoughts — improves reasoning that happens entirely inside the
model, over information already given in the prompt. None of them help with
a different, very common problem: the model doesn't actually know the
answer, because the answer depends on something outside its training data —
today's weather, a specific row in a database, the result of a calculation
too large to do reliably in its head. For that, the model needs to *do*
something, not just think harder.

ReAct, from Yao et al.'s 2022 paper "ReAct: Synergizing Reasoning and
Acting in Language Models," is a prompting pattern for exactly this case.
It structures the model's output as a repeating loop of three parts:
**Thought**, where the model reasons in natural language about what it
knows so far and what it needs to find out next; **Action**, where it
invokes some external capability — a web search, a calculator, a database
query, an API call — rather than guessing; and **Observation**, where the
actual result of that action is inserted back into the context before the
next Thought begins. The loop repeats, each Thought building on the real
Observation that preceded it, until the model has what it needs to produce
a final answer.

## A worked trace

Consider the question: "How much older is the current Formula 1 world
champion than the current F1 constructors' champion team's lead
driver?" A model answering this directly from memory would be guessing at
facts that change yearly and that it may not have been trained on at all.
A ReAct trace instead looks like this:

```
Thought: I need to find out who the current F1 drivers' champion is.
Action: search("current Formula 1 drivers' champion")
Observation: [search result naming the champion and their birth date]
Thought: Now I need the lead driver of the current constructors' champion team.
Action: search("current Formula 1 constructors' champion team lead driver")
Observation: [search result naming that driver and their birth date]
Thought: I have both birth dates. I can now compute the age difference.
Action: calculate(difference between the two dates)
Observation: [the computed difference]
Thought: I now have enough information to answer.
Final Answer: [stated age difference]
```

Each Thought is short and does one job: decide what's needed next and which
action gets it. The model never has to hold the whole plan in its head
before acting — it figures out the next step, takes it, sees what actually
came back, and only then decides what comes after.

## Why grounding beats planning in the dark

The core benefit over reasoning-only techniques is error correction against
reality, not just against the model's own earlier reasoning. If a model
guesses a fact wrong while reasoning in isolation, nothing in that reasoning
process catches it — the wrong fact just becomes a premise for everything
downstream, however logically sound the rest of the chain is. In a ReAct
loop, every guess that matters gets tested against an actual Observation
before the model builds further on it. If a search comes back with
something unexpected — the champion is someone other than assumed, a
calculation produces an odd result — the next Thought sees that real
information and can adjust, ask a follow-up action, or revise its plan,
rather than silently continuing on a false premise.

This is also why ReAct is the backbone pattern behind most modern
tool-using and agentic LLM systems, whatever framework implements them.
Strip away any particular library's scaffolding — LangChain, a custom
orchestration loop, a provider's native "tool use" API — and the shape
underneath is the same: let the model decide what action to take next,
execute that action for real, feed the real result back in, and let the
model decide the next action in light of what it actually learned. The
framework just handles wiring the Action step to real tools and parsing the
loop; the ReAct pattern itself is a way of structuring the prompt and the
conversation, not a piece of software.

## Where it stops helping

ReAct earns its complexity when a task genuinely requires information or
computation the model doesn't reliably have on its own, and when getting
that information might change the plan partway through. It's overkill for
questions the model can simply answer from what it already knows, where
adding an action loop only adds latency and more places for something to go
wrong, with no benefit — there's nothing ungrounded to correct. A good test
before reaching for ReAct: would a wrong answer here plausibly come from
the model confidently asserting something false about the outside world,
rather than from a reasoning slip over information it already has? If so,
grounding each step in a real observation is the right fix; if the
information is all there in the prompt already, a reasoning-only technique
is the simpler and cheaper tool.
