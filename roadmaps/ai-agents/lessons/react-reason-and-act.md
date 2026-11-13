---
title: "ReAct: reasoning and acting"
description: "Interleaving explicit reasoning with tool calls and their results, so each observation reshapes the next thought instead of planning blind."
roadmap: ai-agents
node: react-reason-and-act
group: reasoning-and-planning
video:
  provider: youtube
  id: "Eug2clsLtFs"
  start: 0
  end: null
creator:
  name: Sam Witteveen
  channel_url: https://www.youtube.com/@samwitteveenai
duration_seconds: 900
level: intermediate
prerequisites: [chain-of-thought-prompting]
takeaways:
  - "ReAct interleaves three repeating parts — Thought, Action, Observation — instead of reasoning fully upfront or acting without any reasoning at all."
  - Each observation from a tool call becomes new information the next thought conditions on, so the agent's plan can change mid-task as it learns facts it did not have at the start.
  - This loop is what lets an agent handle tasks where the full plan cannot be known in advance, because some needed information only exists after an earlier step has already run.
  - "ReAct is a direct extension of chain-of-thought: the reasoning text is the same kind of intermediate working memory, now interleaved with real tool results instead of being purely self-generated."
check_questions:
  - q: What is the core difference between ReAct and plain chain-of-thought prompting?
    a: Chain-of-thought reasons entirely inside the model's own generated text with no outside input, while ReAct interleaves that reasoning with real actions and their observed results, so each new thought is conditioned on genuine information pulled from the world rather than only on the model's prior guesses.
  - q: Why can't an agent answering a question like the World Cup population example just plan all its steps in advance?
    a: The identity of the winning country is not known until the first tool call returns, and the two largest cities and their populations are not known until later calls return, so each step's target depends on a fact only the previous observation supplied, making a fixed upfront plan impossible.
deeper: []
related_formulas: []
---

## Three parts, repeated until done

ReAct names a loop with three recurring pieces. A **Thought** is a short piece of reasoning text, written in the same spirit as chain-of-thought, that decides what to do next given everything known so far. An **Action** is a concrete step taken in the world — almost always a tool call, like a search query or an API request — chosen because the preceding thought decided it was the useful next move. An **Observation** is the real result that action returned, inserted back into the model's context. The loop then repeats: the next Thought is generated with that Observation now visible, so it reasons over genuinely new information rather than only its own prior guesses. The loop ends when a Thought concludes the agent has enough to answer the original question.

This is a direct extension of the previous lesson's mechanism. Chain-of-thought showed that a model's own written reasoning becomes real input to its next step. ReAct adds a second source of real input alongside it: actual facts returned from acting on the world, which the model could not have generated correctly on its own.

## A worked trace

Take the question: *What's the population difference between the two largest cities in the country that won the most recent World Cup?*

No single tool call answers this. The agent has to discover the winning country, then its two largest cities, then each city's population, then subtract — and it cannot know which country, or which cities, until earlier steps resolve. Here is a plausible trace:

**Thought 1:** I need to know which country won the most recent World Cup before I can do anything else.
**Action 1:** search("most recent World Cup winner")
**Observation 1:** Argentina won the 2022 World Cup.

**Thought 2:** Now I need the two largest cities in Argentina by population.
**Action 2:** search("largest cities in Argentina by population")
**Observation 2:** Buenos Aires (~3.1 million, city proper) and Córdoba (~1.4 million) are the two largest.

**Thought 3:** I have the two cities. I need precise enough population figures to compute a difference.
**Action 3:** search("Buenos Aires population Córdoba Argentina population")
**Observation 3:** Buenos Aires: approximately 3,120,000. Córdoba: approximately 1,430,000.

**Thought 4:** I now have both populations. The difference is 3,120,000 − 1,430,000 = 1,690,000. I can answer.
**Final answer:** The population difference between Buenos Aires and Córdoba is approximately 1.69 million.

Notice that Thought 2 could not have been written before Observation 1 existed — "Argentina" only enters the agent's context because the first search returned it. Thought 3 similarly depends on the specific city names Observation 2 supplied. Each thought is doing real work reacting to new information, not just restating a plan made at the start.

## Why interleaving beats planning fully upfront

An agent that tried to write a complete plan before taking any action would have to guess the winning country, the two cities, and both populations all at once — exactly the information it does not yet have. Chain-of-thought alone cannot fix this, because no amount of internal reasoning manufactures facts the model was never given. ReAct's interleaving solves this by shrinking the planning horizon to one step at a time: decide the next single action, take it, absorb what comes back, then decide again. This also gives the agent a natural way to recover from a dead end — if Observation 2 had returned unreliable or conflicting data, Thought 3 could have reacted by issuing a clarifying search instead of plowing ahead on bad information, something a rigid upfront plan has no mechanism for.

## Where this leads next

ReAct assumes a single, mostly linear thread of thought-action-observation steps. Some problems do not fit that shape — there are points where several different next actions all look plausible and the agent genuinely cannot tell in advance which will pan out. That is the gap the next lesson addresses, by letting an agent explore more than one reasoning path at once instead of committing to a single chain, and backtracking when a branch turns out to be weak.
