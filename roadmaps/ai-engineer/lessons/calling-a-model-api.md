---
title: Calling a model API
description: Messages and roles, the request and response envelope, tokens in and out, and the error handling every production call needs.
roadmap: ai-engineer
node: calling-a-model-api
group: working-with-models
video:
  provider: youtube
  id: "17FGdx9WHo4"
  start: 0
  end: null
creator:
  name: Telusko
  channel_url: https://www.youtube.com/@Telusko
duration_seconds: 922
level: beginner
prerequisites: [how-llms-work]
takeaways:
  - A model call is an ordinary HTTP POST with a JSON body; an SDK is a convenience over that, not a different mechanism.
  - The request carries a list of messages with roles, and the response carries the completion plus a usage block that is your billing record.
  - Model APIs fail in ways ordinary APIs do not — rate limits, overload, truncation at the token ceiling — and each needs a different response.
check_questions:
  - q: What is actually sent when you pass a conversation to a chat API?
    a: A JSON array of messages, each with a role and content, re-sent in full on every call. The server holds no conversation state between requests.
  - q: Why is retrying a 429 with a fixed delay a bad strategy?
    a: Every client retrying on the same schedule synchronises the load and re-creates the spike. Exponential backoff with jitter spreads the retries out, and the response's retry hint should be honoured when present.
deeper:
  - label: "Harish Neel — LLM Fundamentals Crash Course (tokens, prompting, APIs, tool calls)"
    url: https://www.youtube.com/watch?v=YHNi88FaGug
related_formulas: []
---

## The shape of a call

Strip away the SDK and a model call is just a POST with a JSON body: a model
name, a list of messages, some parameters. The response comes back with the
generated content, a reason it stopped, and a usage block counting input and
output tokens. That's the whole surface.

It's worth seeing the request body in full once. A `model` field names which
model to call. A `messages` array holds entries, each with a `role`
(`system`, `user`, `assistant` or `tool`) and `content`. Optional fields set
temperature, maximum output tokens, and tool definitions. The response
mirrors this shape back: a `choices` or `content` field with the generated
text, a `finish_reason` (or `stop_reason`) telling you whether it stopped
naturally, hit a length limit, or got cut off for a policy reason, and a
`usage` object with `prompt_tokens` and `completion_tokens` — your debugging
tool and your billing record in one place.

## Roles, and who owns the history

The roles matter more than they look. A **system** message sets standing
instructions that persist across the whole conversation. **User** messages
carry the human's input. **Assistant** messages are the model's own previous
turns, and here's the part that surprises people: you send these back
yourself on every single call. The server remembers nothing between
requests. It's less like a phone call and more like a conversation by
letter, where every reply has to quote the entire exchange so far, because
the person on the other end has no memory of writing the last one.

A **tool** role carries a tool call's result back to the model — the
mechanism the [tool calling](/roadmaps/ai-engineer/tool-calling) lesson
builds on. The practical consequence: your application is the sole keeper of
conversation state. Drop a message while reconstructing history, and the
model's apparent memory of that exchange disappears with it.

## Streaming, and why it exists

A call can ask for the response token by token, as a stream of small chunks,
instead of waiting for the whole thing and getting it all at once. Streaming
doesn't make generation faster — the total time to the last token is
unchanged — but it makes the first visible output arrive far sooner, which is
what a user actually perceives as speed. Most conversational interfaces
stream for this reason alone. The cost is implementation complexity: a
streamed response has to be assembled incrementally, and partial JSON from a
structured-output call isn't valid JSON until the stream completes.

## Raw HTTP versus the SDK

The video shows the same call twice: raw HTTP with hand-built headers and
JSON, then the official SDK. That's worth taking seriously, because the SDK
isn't doing anything the raw call couldn't — it's handling authentication
headers, retries, and response parsing for you. Knowing the raw shape means
you can debug a failure the SDK's error message obscures, and it means
switching providers is a matter of mapping one JSON shape to another, not
learning an entirely new mental model.

## What the video doesn't cover: the failure modes

This is where production time actually goes, so here's the detail the video
skips.

**Rate limits** arrive as HTTP `429` and are usually two separate ceilings —
requests per minute and tokens per minute — so you can breach the second
while nowhere near the first. The fix is **exponential backoff with
jitter**: wait roughly `base * 2^attempt` seconds before retrying, with a
small random offset so many clients retrying at once don't all land on the
same instant and recreate the very spike that triggered the limit. Honor the
provider's `retry-after` header when one is sent — it's a direct instruction,
not a guess.

**Overload and transient 5xx errors** are normal at scale, not exceptional,
and should be retried. But only when the call is safe to repeat — an
idempotent read, not an action with side effects — and with a hard cap on
attempts.

**Truncation** is the quiet one. When generation hits the output token
ceiling, the response still comes back as a technically successful call,
with a `finish_reason` of `length` instead of the normal `stop`. The JSON or
structured content you were parsing is now cut off mid-object. Checking the
stop reason on every response, not just the content, is the single cheapest
bug-prevention habit available here.

**Timeouts** need two different values: generous for the call itself, since
a long generation can legitimately take tens of seconds, and short for a
health check sitting in front of it, so a slow model doesn't make your whole
service look down.
