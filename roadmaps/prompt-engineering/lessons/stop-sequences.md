---
title: Stop sequences
description: A string that halts generation the instant it appears — for closing structured output cleanly, stopping role drift, and cutting cost.
roadmap: prompt-engineering
node: stop-sequences
group: output-control
video:
  provider: youtube
  id: "_twqnmtBVMw"
  start: 0
  end: null
creator:
  name: Amit Thinks
  channel_url: https://www.youtube.com/@Amit.Thinks
duration_seconds: 89
level: beginner
prerequisites: [output-length-control]
takeaways:
  - A stop sequence halts generation the instant an exact string is produced, while max_tokens halts generation after a fixed count regardless of content.
  - Stopping on a structural marker, like a closing brace or a turn delimiter, prevents unwanted trailing content from ever being generated, which is cheaper than generating and discarding it.
  - Stop sequences only match exact strings, so they work for structural boundaries, not for semantic conditions such as stopping once the main point has been made.
check_questions:
  - q: How does a stop sequence differ from max_tokens?
    a: A stop sequence is content-triggered — it halts generation the instant a specific string appears — while max_tokens is count-triggered and halts after a fixed number of tokens regardless of content.
  - q: Why can't a stop sequence express a condition like "stop once the main point is made"?
    a: A stop sequence only matches an exact string; it has no way to evaluate the meaning or completeness of what's been generated so far.
deeper:
  - label: "Champion Edtech — Parameters in Playground 3: Stop sequences"
    url: https://www.youtube.com/watch?v=95EElkUcfUQ
related_formulas: []
---

A stop sequence is a string you give the API in advance; the moment the model generates
that exact string, generation halts immediately, and (depending on the provider) the
triggering string itself is usually excluded from what you get back. It's a small,
specific tool, and it's easy to overlook because the UI for it is often just one plain
text field — but it solves a few problems that nothing else does cleanly.

## A trigger, not a limit

The thing to hold onto is that a stop sequence is *content-triggered*, not
*count-triggered*. `max_tokens` stops generation after a fixed number of tokens no matter
what the content is; a stop sequence stops generation the instant a specific piece of
content appears, no matter how many tokens that took. They solve different problems and
are often used together: a generous `max_tokens` as a safety ceiling, and one or more
stop sequences for the actual, content-aware cutoff you care about.

## Closing a format cleanly

If you're asking a model to produce a single JSON object and nothing else, a stop
sequence on the closing delimiter — a literal `}` at the end of expected output, or a
custom marker like `<<END>>` that you instruct the model to emit right after the object —
stops generation the instant the structure is complete. Without it, some models will
continue past a perfectly valid JSON object with an explanation, a follow-up question, or
a second example you didn't ask for, all of which you'd otherwise have to detect and
strip out after the fact. The stop sequence prevents that content from ever being
generated in the first place, which is strictly better than generating it and discarding
it — you pay nothing in tokens or latency for text you were going to throw away anyway.

## Stopping the model from writing both sides of a conversation

In a chat-style prompt, especially one built from a raw text completion rather than a
chat-specific endpoint, a model will sometimes keep going past its own turn and start
writing what it predicts the *other* speaker would say next — inventing a "User:" turn,
answering its own invented question, and continuing the back-and-forth indefinitely.
Setting a stop sequence on the turn markers you're using (`"\nUser:"`, `"\nHuman:"`,
whatever delimiter your prompt format uses) stops generation the instant the model starts
drifting into the next turn, which is both a correctness fix — you don't want a
fabricated user message in your output — and the same cost benefit as above.

## Cost and latency, almost for free

Every token a model generates costs money and time, including tokens you're going to
delete. Because a stop sequence halts generation immediately rather than after the fact,
it's one of the few optimizations that reduces both cost and latency with essentially no
downside — there's no tradeoff to weigh, as there is with, say, choosing a smaller model.
The catch is that it only helps for content you can describe exactly: a stop sequence has
to be an exact string match, so it's useful for structural boundaries (closing brackets,
turn markers, section headers) and not for semantic conditions like "stop once you've
made your main point," which no stop sequence can express.

## Stop sequences versus max_tokens

Use a stop sequence whenever you can name the exact string that marks "this response is
done" — it's cheaper, more precise, and never truncates mid-thought the way a token limit
can. Keep `max_tokens` in place regardless, as a backstop for the case where your stop
sequence never appears (a malformed response, an edge case your prompt didn't anticipate)
and generation would otherwise run unbounded. The two together — a content-triggered stop
for the common case, a count-triggered ceiling for everything else — cover the two
failure modes neither one covers alone.
