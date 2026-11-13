---
title: What are tools?
description: Giving a model a function it can call turns it from a text generator into something that can fetch real information and act in the world.
roadmap: ai-agents
node: what-are-tools
group: tools
video:
  provider: youtube
  id: "ATUUd2bpRfo"
  start: 0
  end: null
creator:
  name: Lucas Soares | Automata Learning Lab
  channel_url: https://www.youtube.com/@automatalearninglab
duration_seconds: 1200
level: intermediate
prerequisites: [react-reason-and-act]
takeaways:
  - A tool is a function the model can choose to invoke mid-conversation, which lets it read live data or take an action instead of answering only from what it learned during training.
  - The model never runs the tool itself — it emits a request to call one, your code executes it, and the result is fed back into the conversation as new context.
  - Tools close the gap between "knows about the world as of training" and "can find out what's true right now," which is what turns a chatbot into an agent.
  - Choosing the right tool and the right arguments is itself a reasoning step, not a mechanical lookup, which is why tool use builds directly on the ReAct loop.
check_questions:
  - q: Why can't a language model answer "what's the weather in Lisbon right now" from its own weights, no matter how good the model is?
    a: The model's weights encode patterns learned from training data with a fixed cutoff, not a live connection to the present — weather changes by the hour, so no amount of training can bake in a fact that didn't exist yet when training happened.
  - q: When a model "calls a tool," what is actually happening at the software level?
    a: The model outputs a structured request naming a function and its arguments; the surrounding application code — not the model — executes that function, and the application then feeds the function's return value back into the model's context so it can use it.
deeper:
  - label: "I Built an AI That Fetches Weather, Crypto & News — Pavan Belagatti"
    url: https://www.youtube.com/watch?v=90e9rR1loZs
related_formulas: []
---

## A model that can't check the weather

Ask a plain language model what the weather is in Lisbon right now, and it faces a
problem no amount of training fixes: it has no idea. Not because weather is hard to
reason about, but because the model's knowledge is frozen at whatever point its
training data was collected, and weather is a fact about this exact moment. The model
can describe what Lisbon's climate is typically like in October, it can explain how
weather forecasting works, it can even produce a plausible-sounding number — but that
number would be a guess dressed up as an answer, not information. This is the wall
every pure text generator hits the moment a question depends on something that
changes after training ends, or that was never public text in the first place: a
user's private calendar, a company's internal database, the current price of a stock.

A tool is how the model gets past that wall. Instead of answering from memory alone,
the model is given the ability to call a function — `get_weather(city)`, say — that
actually goes and fetches the current temperature in Lisbon from a real weather
service. The model doesn't run this function itself; it asks for it to be run, waits
for the answer, and then writes its final response using that answer as grounding.
The difference in outcome is total: one answer is a hallucination with good grammar,
the other is a fact.

## The mechanics: who does what

It helps to be precise about the division of labor, because it's easy to picture the
model as "reaching out" to the internet, which it never literally does. Three things
happen in sequence. First, your application tells the model which tools are
available — in the weather example, a function named `get_weather` that takes a
`city` argument. Second, when the model decides a tool would help answer the current
question, it doesn't call the function directly; it emits a structured message saying,
in effect, "call `get_weather` with `city="Lisbon"`." Third, your own code — not the
model, which has no network access and no runtime of its own — actually executes that
function, gets back something like `{"temp_c": 17, "condition": "cloudy"}`, and hands
that result back to the model as a new piece of context. Only then does the model
write the sentence the user actually reads: "It's 17°C and cloudy in Lisbon right
now."

This loop — propose a call, execute it outside the model, feed the result back in — is
the same shape as the Act step in ReAct, which this lesson builds directly on. ReAct
gave the model a way to interleave reasoning with action in general; tools are the
concrete mechanism that makes "action" something a model can actually trigger in a
real system rather than just narrate.

## Tools aren't only for fetching facts

Weather is the easy, almost clichéd example, but the same mechanism covers two
distinct categories of capability that are worth telling apart. **Information
tools** retrieve something the model doesn't already know: a search engine query, a
database read, a stock price, a user's order history. **Action tools** change
something in the world: sending an email, creating a calendar event, placing an
order, writing a row to a database. Both are "just functions" from the model's point
of view — it names one and supplies arguments either way — but the stakes differ
sharply. Getting a weather lookup wrong wastes a retry. Getting an "send this email"
call wrong sends a real email. That asymmetry is why later lessons in this group spend
real time on validating arguments and handling failures before a tool call executes,
rather than treating every tool as equally low-risk.

## Why this is the hinge of "agent" versus "chatbot"

A system that only ever produces text, however fluent, is bounded by what was in its
training data. The moment you hand it even one working tool, that boundary moves: it
can now check something, look something up, or do something, and incorporate the
result into how it continues. That's not a minor feature addition — it's the
definitional difference this roadmap has been building toward since its first
lesson. "Agent" doesn't mean "bigger model" or "longer prompt." It means a system that
can take actions that have consequences outside its own context window, and tools are
the literal mechanism by which that happens. Every later lesson in this group —
writing good tool definitions, wiring them into a provider's API, handling the ones
that fail — is about making that one idea, "the model can call a function," work
reliably instead of just working in a demo.

## A small worked example

Picture a travel assistant answering "should I pack a coat for my trip to Lisbon next
week?" A pure text model can only hedge: "Lisbon is generally mild in October, but
check the forecast." A tool-using agent instead reasons: *this question depends on a
forecast I don't have, and I have a `get_weather` tool, so I should call it before
answering.* It calls `get_weather(city="Lisbon", date="next week")`, gets back
something like `{"temp_c": 14, "condition": "rain"}`, and only then produces its real
answer: "Yes — it's expected to be around 14°C with rain, so bring a coat." The
quality of that final sentence depends entirely on a tool call that happened in the
middle, invisible to the user, where the model stopped generating text for a moment
and reached for a fact instead.
