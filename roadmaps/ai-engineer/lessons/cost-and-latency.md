---
title: Cost and latency
description: Token economics, time to first token, caching, routing and batching — the levers that decide whether an LLM feature is affordable.
roadmap: ai-engineer
node: cost-and-latency
group: production
video:
  provider: youtube
  id: "_nmlHbSB8kM"
  start: 0
  end: null
creator:
  name: OpenAI
  channel_url: https://www.youtube.com/@OpenAI
duration_seconds: 1384
level: advanced
prerequisites: [prompt-caching]
takeaways:
  - Output tokens cost several times what input tokens cost, because each one is a separate forward pass while the prompt is processed in parallel.
  - The unit that matters is cost per completed task, not cost per token — a cheap model that needs three attempts is not cheap.
  - Perceived latency is governed by time to first token, which streaming addresses without making the total any shorter.
check_questions:
  - q: Why do output tokens cost more than input tokens?
    a: Prefill processes the entire input in one parallel pass, while each output token requires its own forward pass over the whole model with all previous tokens in context. The work per token is simply greater.
  - q: Why can streaming improve a product without improving its latency?
    a: Total generation time is unchanged, but the user sees the first words almost immediately instead of waiting for the whole response. Time to first token is what a person experiences as speed.
deeper:
  - label: "Applied AI Conf — How prompt caching cut our LLM bills in half"
    url: https://www.youtube.com/watch?v=BtomaNlH6NI
  - label: "Mark Moyou — Mastering LLM inference optimisation"
    url: https://www.youtube.com/watch?v=9tvJ_GYJA-o
related_formulas: [littles-law, arithmetic-intensity, memory-bound-token-latency, kv-cache-size]
---

## The asymmetry everything follows from

Input and output tokens aren't priced alike, and the reason is mechanical,
not just commercial — it follows directly from the prefill-and-decode split
covered in [how LLMs work](/roadmaps/ai-engineer/how-llms-work). The prompt
gets processed in a single parallel pass across every position at once.
Every output token, by contrast, needs its own separate sequential forward
pass through the entire network, attending over everything generated before
it. It's simply far more computational work per token produced than per
token merely read.

A typical pricing split has output tokens costing three to five times as
much per token as input tokens, directly reflecting that difference in
compute cost. Two things follow from this. Long prompts are cheaper than
they intuitively feel — a generously detailed prompt costs less than the
same volume of generated output would, though stuffing context still costs
accuracy for reasons covered in the
[context window](/roadmaps/ai-engineer/the-context-window) lesson,
independent of dollar cost. And verbose output is genuinely expensive, which
makes an explicit instruction toward conciseness a real cost control
measure, not just a stylistic preference.

## A worked end-to-end example

Consider a feature handling ten thousand requests a day, each averaging two
thousand input tokens and five hundred output tokens, against a hypothetical
price of three dollars per million input tokens and fifteen dollars per
million output tokens.

Input cost per day: ten thousand requests times two thousand tokens, divided
by a million, times three dollars — sixty dollars. Output cost: ten thousand
times five hundred, divided by a million, times fifteen dollars —
seventy-five dollars. Total: a hundred and thirty-five dollars a day, close
to four thousand dollars a month, for a feature that might feel small in a
demo.

Double the input through a longer system prompt or more retrieved context,
and you roughly double the input half of that bill. Double the average
output length through verbose, unconcise responses, and you roughly double
the larger output half. That's exactly why trimming output length is
usually the bigger lever of the two for a text-heavy feature.

## The unit that matters: cost per completed task

The number worth optimizing is cost per completed task, not cost per token
or even cost per call. An agent that costs a fraction as much per individual
call but needs four attempts to actually succeed hasn't saved anything
relative to a more expensive agent that succeeds on its first attempt — it
may well have cost more overall, while also taking longer. This is the same
reasoning that makes [evals](/roadmaps/ai-engineer/why-evals-matter)
essential to any genuine cost comparison: a cheaper model that needs
substantially more retries or scaffolding to reach the same success rate
isn't actually a cheaper system once that overhead gets counted honestly.

## Levers, roughly in order of typical return

**Caching**, covered in its own
[lesson](/roadmaps/ai-engineer/prompt-caching), is usually the largest and
easiest win available — arrange prompts so the stable portion comes first
and can be served from cache on repeat calls.

**Routing** sends most requests, which in most real systems are genuinely
easy, to a small, cheap model, escalating only the ones that actually need a
more capable model. The escalation decision is itself a classification
problem a small model can often make reliably on its own.

**Shortening output** through structured responses instead of unconstrained
prose — no restated question, no unnecessary preamble — directly attacks the
more expensive half of the pricing asymmetry.

**Batching**, where latency isn't user-facing — overnight processing, bulk
classification jobs, large backfills — trades a longer turnaround time for a
substantial provider discount on bulk asynchronous processing.

**Model cascades** formalize the routing idea further: try the cheapest
model first, check its own confidence or validate its output against some
cheap criterion, and escalate to a progressively more capable model only
when that check fails, instead of deciding up front which single model every
request will use.

## Little's Law, applied to concurrent requests

Little's Law — see the [full formula](/formulas/littles-law) — states that
the average number of requests concurrently in a system equals the average
arrival rate multiplied by the average time each request spends in the
system. It's the same logic as a coffee shop line: if customers arrive every
six seconds and each takes three minutes to serve, there are always thirty
people in the shop at once, not one.

Applied to an LLM service: ten requests per second, each taking an average
of three seconds to complete, means thirty requests in flight at any given
moment — not ten. This directly sets the concurrency a serving
infrastructure must actually be provisioned to handle, and it's also exactly
why reducing a feature's per-request latency, even with the arrival rate
unchanged, directly reduces the infrastructure capacity needed to serve that
same traffic.

## Time to first token, and what streaming actually fixes

What a user perceives as "speed" is dominated far more by **time to first
token** — how long an interface sits visibly empty before anything appears —
than by the total time the complete response takes to finish. Time to first
token is dominated by the prefill phase, so it grows directly with prompt
length. That's yet another reason caching matters here specifically: a cache
hit skips most of prefill, and the first visible token can appear almost
immediately after.

**Streaming** the response token by token as it's generated is the standard
remedy, and the honest framing is that it changes a user's *perception* of
elapsed time, not the actual total duration. Total generation time is
completely unchanged by streaming — only the long, empty wait before
anything visible happens disappears.

In an agent making several sequential tool calls before producing any
visible output, this compounds in a way streaming can't help with: ten
sequential tool calls at two seconds each is twenty full seconds before the
first useful word can even begin streaming, because there's simply nothing
yet to stream. Parallelizing independent tool calls and cutting the total
number of sequential iterations an agent needs matters far more there than
anything to do with the model's own raw generation speed.
