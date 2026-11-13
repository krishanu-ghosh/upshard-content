---
title: Tracing and observability
description: Why LLM traces are a different problem from uptime monitoring, what a span should carry, and finding the failing step in an agent run.
roadmap: ai-engineer
node: tracing-and-observability
group: evaluation
video:
  provider: youtube
  id: "XBaznoTRDFI"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1243
level: advanced
prerequisites: [building-an-eval-set, the-agent-loop]
takeaways:
  - A trace is the whole run as a tree of spans, and for an LLM system each span must carry the full prompt and response, not just timing.
  - Traditional monitoring asks whether the system responded; LLM observability asks whether the response was any good, which no status code reports.
  - Production traces are where the next eval cases come from, which makes tracing the input to evaluation rather than a separate concern.
check_questions:
  - q: Why is a 200 response close to meaningless for an LLM feature?
    a: The call succeeded and says nothing about whether the answer was grounded, relevant or safe. Every failure that matters here returns 200 with fluent text.
  - q: Why must spans record full prompts and responses rather than just latency?
    a: Because the investigation is almost always about content — what was retrieved, what the model was actually sent, what a tool returned. Timing alone cannot reconstruct why an agent went wrong.
deeper:
  - label: "Will Velida — Distributed tracing for multi-agent systems with OpenTelemetry"
    url: https://www.youtube.com/watch?v=28z8UyjpjAk
  - label: "AI Engineer — Everything you need to know about agent observability"
    url: https://www.youtube.com/watch?v=-aM2EDTiaMs
related_formulas: []
---

## A different question than uptime

Conventional service observability asks whether the system is up, fast, and
free of errors. An LLM feature can score perfectly on all three and still be
useless in practice — confidently answering from the wrong document, drifting
off the intended tone, or quietly abstaining on a question it should be able
to handle. None of this shows up in a latency histogram or an HTTP error
rate, because every one of these failures returns a successful response with
fluent, well-formed text.

The signals that actually matter here are different in kind. What was
retrieved for a given query, and did it genuinely contain the answer? What
exactly was present in the context window at generation time? Which tools
were called, and precisely what did they return? What did the model actually
produce at each intermediate step of a multi-step run? How many tokens and
how much money did a given run consume? And did the user who received the
result accept it, or immediately try again?

## OpenTelemetry conventions for generative AI

A trace represents one complete run as a tree of nested **spans**: a
top-level span for the overall request, with child spans beneath it for each
model call, each retrieval, each tool execution, and each sub-agent
invocation — nested exactly where each one logically belongs in the flow.
Standard semantic conventions have emerged for naming these spans and their
attributes consistently. A model-call span typically carries the model name
and parameters used, the full rendered prompt and response, and token counts
broken out by input and output. A retrieval span carries the query issued
and the ids and relevance scores of whatever came back. A tool span carries
the arguments passed in and the raw result returned.

Adopting these shared conventions, rather than inventing a bespoke logging
format, means an LLM-specific trace can live inside the exact same
observability system already used for the rest of a service's
infrastructure — not a separate, parallel tool just for this one part of the
stack.

## Sampling at volume, and what it costs you

A full trace, carrying entire prompts and responses verbatim, can run to many
kilobytes per run — multiple orders of magnitude larger than a typical HTTP
access log line. That makes capturing every production run expensive at real
volume. Standard sampling strategies built for ordinary web traffic, like
keeping a fixed small percentage of requests at random, can quietly discard
exactly the rare failure cases that matter most for debugging: a rare failure
mode by definition appears in only a small fraction of traffic, so a random
sample can simply miss it. A better approach for LLM systems samples
adaptively — always keep traces that hit an error, that received explicit
negative user feedback, or that triggered a guardrail, and sample only a
small baseline fraction of the otherwise unremarkable, successful runs.

## Cost attribution, concretely

Every span already carries token counts and a known price per token, so a
complete trace can be rolled up into an actual dollar cost for that run. That
cost can then be attributed further: by which user made the request, by
which feature or endpoint handled it, by which model was called. This turns
"our LLM costs rose last week" from a vague observation into a specific,
answerable question — which feature's traces grew in volume, length, or
model tier, and is that growth something to welcome or something to fix.

## From trace to eval: the loop that closes

A production trace isn't only useful for debugging a single incident after
the fact. It's a real input with a real, observed outcome attached, which is
exactly the raw material the
[eval set](/roadmaps/ai-engineer/building-an-eval-set) is built from.
Treating tracing as the primary feeder for evaluation — regularly pulling
real traces, especially the ones with negative outcomes, into the eval set
as new test cases — closes the loop between what happens in production and
what gets measured going forward. Teams that use tracing this way tend to
improve steadily over time. Teams that treat tracing only as a dashboard to
glance at during an incident don't see the same compounding benefit.

## A debugging narrative, to make this concrete

A user reports that an agent gave a wrong answer about their account balance.
Opening the trace for that run shows the retrieval span first: it returned
the correct account document, so retrieval wasn't the problem. The next span
shows the context-assembly step included that document alongside four other
unrelated documents from an overly broad initial search, pushing the correct
document into the middle of a long prompt. The final generation span shows
the model's answer drawing a number from one of the unrelated documents
instead of the correct one — a direct, visible instance of the
lost-in-the-middle effect covered in the
[context window](/roadmaps/ai-engineer/the-context-window) lesson, now caught
concretely in a real trace rather than inferred from research.

The fix follows directly from this trace: tighten the retrieval and
re-ranking step so fewer, more precisely relevant documents reach the prompt.
That fix would have been close to impossible to arrive at correctly without
seeing the full span-by-span trace of exactly what the model was given at
each stage.
