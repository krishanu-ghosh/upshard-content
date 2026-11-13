---
title: The agent loop (advanced)
description: Plan, act, observe, repeat — owning the control loop, deciding when to stop, and why agents stall or run away.
roadmap: ai-engineer
node: the-agent-loop
group: agents
video:
  provider: youtube
  id: "8kMaTybvDUw"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1025
level: advanced
prerequisites: [tool-calling, rag-pipeline]
takeaways:
  - The loop itself is four lines of code; everything that makes an agent reliable lives in what you do between the iterations.
  - Every loop needs explicit termination on several axes — iterations, tokens, wall-clock time and cost — because a model will not reliably decide to stop.
  - Owning the loop rather than inheriting it from a framework is what makes an agent debuggable when it behaves strangely.
check_questions:
  - q: Why does an agent loop need a stopping condition beyond the model saying it is done?
    a: Because the model may never say so. It can alternate between two tools, retry a failing call indefinitely, or convince itself more work is needed. Budgets on iterations, tokens, time and cost are the only guarantee.
  - q: What makes a looping agent hard to debug compared with a pipeline?
    a: The failure is usually several steps upstream of where it surfaces, and the run is not reproducible. Without a recorded trace of every prompt, call and result, you are guessing.
deeper:
  - label: "Kam Lasater — Let's build an agent from scratch"
    url: https://www.youtube.com/watch?v=xzXdLRUyjUg
  - label: "Yogendra Miraje — Planning agents without losing control"
    url: https://www.youtube.com/watch?v=sl3icG-IjHo
related_formulas: []
---

## Four lines, then the hard part

Here's the whole loop. Call the model with the conversation and the available
tools. If it asks for a tool, run it. Append the result to the conversation.
Repeat until it answers directly or you stop it. That's four lines, and
writing them takes minutes.

The hard part lives in the gaps between those lines. What exactly goes into
the context on iteration twelve? What happens when a tool fails twice in a
row? How do you detect that the agent is alternating uselessly between the
same two calls, and when do you decide it has had enough attempts? None of
that is in the loop itself. All of it determines whether the agent actually
works.

## Memory: short-term context versus long-term storage

An agent's **short-term memory** is simply whatever is currently in its
context window: the conversation so far, including every tool call and result
from the current run. It disappears the moment the run ends, unless something
explicitly saves it. **Long-term memory** is state deliberately persisted
outside the context window — a file, a database row, a note — that can be
read back in on a future run, or even by a different agent entirely.

The [context engineering](/roadmaps/ai-engineer/context-engineering) lesson's
"write" move is exactly this: moving information from the temporary
short-term space into something durable before compaction or the run simply
ending would otherwise lose it.

## Explicit planning versus step-by-step improvisation

Some agent designs ask the model to produce an explicit plan up front — a
numbered list of sub-tasks — before executing any of them, then work through
that plan largely as written, adjusting only when a step fails outright. This
produces more coherent, legible traces and is easier to debug. The cost is
speed: when an early step reveals that the plan itself was wrong, the agent
is slower to adapt.

Other designs have the model decide one step at a time with no plan at all,
reacting purely to what the previous step returned. That's more adaptive to
genuine surprises, but carries a real risk of local, myopic decisions that
never add up to coherent progress. A **reflection** step — the agent pausing
periodically to critique its own progress against the original goal before
continuing — is a common middle ground for either design, catching drift
before it compounds.

## A worked trace, narrated

Consider an agent asked to find and summarise a customer's recent support
history. Step one: it calls a lookup tool with the customer's email and gets
back a customer ID. Step two: it calls a tickets tool with that ID and gets
back ten ticket identifiers with no detail. Step three: realising it needs
the actual ticket content, it calls a ticket-detail tool, but gets back a
result for the wrong ticket — a subtle identifier format mismatch between the
two tools. Step four: the model notices the mismatch, reasons about what went
wrong, and retries with a corrected identifier format. Step five: now holding
the right ten tickets' content, it writes a summary and answers.

Nothing about this sequence was specified in advance. The length, the detour
in step three, and the self-correction in step four all emerged from the
model reasoning about what each result actually contained. That's both the
entire value of an agent over a fixed pipeline, and exactly why a run like
this is hard to predict, test, or bound ahead of time.

## ReAct, and what it actually added

Interleaving explicit reasoning with each action — think about what to do
next, act, observe the result, think again — is usually called the ReAct
pattern. Its real contribution was showing that a model reasoning between
actions noticeably outperforms one that just emits tool calls with no stated
justification, because each observation gets genuinely interpreted before the
next decision rather than pattern-matched straight into the next call. Modern
tool-calling APIs have largely absorbed this behaviour implicitly, which is
why it's rarely implemented as a separate step anymore. But when an agent is
making visibly poor choices, giving it more room to reason between steps —
rather than adding more tools — is still often the right first fix.

## Why agents stall, and why they run away

**Stalling** looks like the same tool called repeatedly with the same or
nearly the same arguments. The cause is almost always that the previous
result carried no genuinely new signal — an empty list, an opaque error with
no actionable detail — leaving the model nothing to condition its next
decision on. The standard fix: detect a repeated call and inject something
different, whether that's a hint, a narrowed scope, or simply a forced stop.

**Running away** is the opposite failure. Each step generates plausible
further work, and the agent never naturally concludes it's done. This is
what explicit budgets are for, and they need to cover several axes at once,
because each catches a different kind of runaway: a cap on iterations, a cap
on total tokens, a wall-clock timeout, and a hard cost ceiling. An agent that
respects three of these four will eventually find the one you forgot to set.

**Losing the plot** comes from the window gradually filling with accumulated
tool output until the original goal sits far back in the context, weighted
only weakly by attention. Compaction, covered in
[context engineering](/roadmaps/ai-engineer/context-engineering), is the
structural remedy. Simply re-stating the original goal near the end of the
context on every iteration costs almost nothing and helps more than its
simplicity suggests.

Record everything the loop does, in full. By the nature of non-determinism,
the specific run you'll need to understand later is one you can't just
reproduce by trying again — which is exactly what
[tracing](/roadmaps/ai-engineer/tracing-and-observability) exists to capture.
