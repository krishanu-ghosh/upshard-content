---
title: Self-critique and reflection
description: "An agent reviewing its own draft or plan against the goal, catching a specific flaw, and revising before continuing or finalizing."
roadmap: ai-agents
node: self-critique-and-reflection
group: reasoning-and-planning
video:
  provider: youtube
  id: "v5ymBTXNqtk"
  start: 0
  end: null
creator:
  name: LangChain
  channel_url: https://www.youtube.com/@LangChain
duration_seconds: 840
level: intermediate
prerequisites: [react-reason-and-act]
takeaways:
  - Self-critique adds a distinct step where an agent re-reads its own draft output or plan against the original goal and names specific flaws, before a separate revision step fixes them.
  - This pattern, often called Reflexion, lets an agent catch its own mistakes without needing a human or an external grader to point them out first.
  - Critique is most useful when it is specific and checkable — naming a concrete gap or error — rather than a vague instruction like "double check your work," which rarely surfaces anything the first pass missed.
  - Self-critique costs an extra generation pass per round, and an agent can loop on it more than once, so it is deployed selectively on outputs where errors are costly rather than on every single response.
check_questions:
  - q: What two distinct steps make up self-critique and reflection, and why does splitting them matter?
    a: A critique step that re-reads the draft against the goal and names a specific flaw, and a separate revision step that fixes that named flaw, and splitting them matters because asking a model to criticize and fix in one breath tends to produce vague, unchecked "looks good" judgments, while forcing an explicit critique first gives the model something concrete to condition the revision on.
  - q: Why is a vague self-check instruction like "double check your work" usually less effective than real self-critique?
    a: A vague instruction gives the model no specific failure mode to test for, so it tends to just restate confidence in the original answer, whereas a critique step that checks the draft against named criteria — did it answer the actual question, did it miss a constraint — surfaces concrete, fixable problems the first pass missed.
deeper: []
related_formulas: []
---

## Catching your own mistakes before anyone else does


Every pattern in this group so far — chain-of-thought, ReAct, tree of thought — improves how an agent arrives at an answer or a plan. None of them guarantee the result is actually correct once it exists. Self-critique and reflection, often called Reflexion in the research literature, adds a step after a first draft exists: the agent re-reads its own output against the original goal, specifically looking for ways it falls short, and only then produces a revision. The key structural choice is that critique and revision are kept as two separate steps rather than folded into one. A model asked to "check your answer and fix it if wrong" in a single instruction tends to glance at its own output, decide it looks fine, and move on — there is no forcing function that makes it actually find anything. Splitting the step in two changes that: the critique pass has exactly one job, which is to name a specific problem with the draft, and the revision pass has exactly one job, which is to fix the problem the critique already named.

## The mechanism: critique against the goal, then revise against the critique

A self-critique step takes three inputs: the original goal or question, the draft produced so far, and (often) a short list of specific failure modes to check for — did the draft actually answer what was asked, did it skip a stated constraint, is a claimed fact actually supported, does a plan account for a dependency it needs. The output of this step is not a new answer; it is a diagnosis, ideally naming one or two concrete issues rather than a general verdict. The revision step then takes the original draft plus that diagnosis and produces a new draft that specifically addresses the named issues, rather than regenerating from scratch. This separation is what makes the pattern reliable: a model is generally much better at judging a fully-formed draft against a checklist than it is at getting everything right in one unexamined pass, for the same underlying reason chain-of-thought works — evaluation against existing written text is a different, easier task than generation from nothing.

## A worked example

Suppose an agent is asked: *Plan a one-day itinerary for a visitor to a city with exactly 6 hours, who wants to see the top historical site and also try the most famous local food, without needing a car.*

**Draft 1:** "9:00 AM: Visit the historical fort (allow 3 hours). 12:00 PM: Taxi to the fort district's famous noodle restaurant for lunch (1 hour). 1:00 PM: Walk to the old town market to browse (1.5 hours). 2:30 PM: Visit the waterfront viewpoint (1 hour)."

**Self-critique:** "The plan uses a taxi at 12:00 PM, but the constraint says the visitor does not want to need a car — a taxi violates that constraint even though it isn't the visitor's own car, since the intent is clearly to avoid needing arranged transport. Also, the total time adds up to 6.5 hours (3 + 1 + 1.5 + 1), which exceeds the stated 6-hour limit by 30 minutes."

Notice what the critique does: it does not say "this plan could be better" — it names two specific, checkable problems, one a constraint violation and one an arithmetic overrun, each traceable to an exact line in the draft.

**Revision:** "9:00 AM: Visit the historical fort (3 hours, walkable from the city center). 12:00 PM: Walk 10 minutes to the fort district's famous noodle restaurant for lunch (1 hour). 1:00 PM: Walk to the old town market, within the same walkable district (1.5 hours). 2:30 PM: End the day with a short walk to the nearby waterfront viewpoint (30 minutes). Total: 6 hours, all on foot."

The revision keeps everything that worked in the first draft — the fort, the noodle restaurant, the market — and changes exactly the two things the critique flagged: it replaces the taxi with a walk by choosing a restaurant within walking distance, and it trims the viewpoint visit from 1 hour to 30 minutes to bring the total back to exactly 6 hours. Neither fix required restarting the plan from zero; both were targeted responses to a named flaw.

## Why this beats hoping the first draft is right

Without a critique step, an agent's only check on quality is whatever confidence is baked into its first generation pass, and a model generating a plan has no particular reason to notice a self-contradiction like "no car" followed immediately by "taxi" unless something explicitly asks it to compare the draft against the stated constraints again. The self-critique step is that explicit re-check, performed as a distinct piece of reasoning with the draft already sitting in context — the same way ReAct's observations give a later thought real information to react to, a critique gives a later revision a real, specific target to fix rather than a vague sense that something might be wrong.

## Where this fits, and its limits

Self-critique is not free: each round costs an additional generation pass, and a thorough implementation might loop critique-then-revise more than once until the critique step finds nothing further wrong. That cost means it is usually applied selectively — on final outputs that matter, on plans with hard constraints, on code before it runs — rather than wrapped around every single response an agent produces. It also inherits a real limit: a model's self-critique is only as good as its ability to notice the flaw in the first place, so it catches constraint violations and internal inconsistencies far more reliably than it catches factual errors the model simply believes to be true. That gap is exactly why agent systems downstream in this roadmap add external checks — human review, automated evaluation, tracing — rather than relying on self-critique alone to catch everything.
