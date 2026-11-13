---
title: Error handling for tool calls
description: Tool calls fail, time out, and return garbage, so a reliable agent needs retries, backoff, and a hard stop — not blind faith that every call succeeds.
roadmap: ai-agents
node: error-handling-for-tool-calls
group: tools
video:
  provider: youtube
  id: "7tjAWFE55tM"
  start: 0
  end: null
creator:
  name: LangChain
  channel_url: https://www.youtube.com/@LangChain
duration_seconds: 300
level: intermediate
prerequisites: [native-function-calling]
takeaways:
  - A tool call can fail in several distinct ways — a timeout, a rate limit, a malformed response, or a result that parses fine but is simply wrong — and each deserves a different response, not one generic catch-and-retry.
  - Retrying with exponential backoff turns a brief, transient failure into a successful call a moment later, while retrying immediately and repeatedly just hammers a struggling service harder.
  - Every retry loop needs a hard ceiling — a maximum attempt count or total time budget — because an agent that retries forever on a permanently broken tool silently burns time and money without ever telling anyone it's stuck.
  - When a tool genuinely can't be made to work, the right move is often to tell the model the tool failed and let it decide whether to try a different tool or give up and inform the user, rather than crashing the whole agent process.
check_questions:
  - q: Why is exponential backoff better than retrying a failed tool call immediately and repeatedly?
    a: An immediate, repeated retry hits a struggling or rate-limited service at exactly the moment it's least able to cope, often making things worse, whereas backoff waits progressively longer between attempts, giving a transient problem time to clear before the next try.
  - q: What's the risk of a retry loop with no maximum attempt count or time budget?
    a: If the underlying failure is permanent rather than transient — a dead endpoint, a bad API key — the agent retries forever, consuming time and API cost indefinitely while never surfacing the problem to anyone who could fix it.
deeper:
  - label: "Why Temporal Beats Standard Retry Logic — Temporal"
    url: https://www.youtube.com/watch?v=3Iw8lRXenR0
related_formulas: []
---

## Tool calls are not guaranteed to work

Everything up to this lesson has assumed a tool call goes through cleanly: the model
asks, your code runs the function, the result comes back, the model continues. Real
tools don't cooperate that reliably. A flight-search API can time out under load. A
weather service can hit its rate limit and reject your request. A database query can
return a result in a shape your code didn't expect. None of these are exotic edge
cases — at any real volume of calls, they are the normal background rate of things
going wrong, and an agent that has no plan for them either crashes outright or, worse,
proceeds as if a failed call had succeeded and gives the user a confidently wrong
answer built on bad or missing data.

## Not every failure is the same failure

Lumping every error into one generic `try/catch` that just retries is a common
shortcut, and it hides important differences. A **timeout** usually means the service
is slow or temporarily overloaded — worth retrying, probably after a short wait. A
**rate limit** means the service is telling you explicitly to slow down — retrying
immediately only makes it angrier; backing off is not optional here, it's the
documented contract. A **malformed or unexpected response** — a field missing, a type
that doesn't match what you expected — often means retrying the exact same call will
fail the exact same way, so the useful response is to log it and stop, not to hammer
it. And a result that parses perfectly but is simply *wrong* — a tool returning a
plausible but incorrect flight price due to a bug on its end — is the hardest case,
because nothing about the shape of the response tells you anything failed at all;
catching this one usually depends on validation logic specific to the tool, not on
generic error handling.

## A worked scenario: the flaky API

Picture an agent that calls a flight-pricing API as part of answering "what's the
cheapest flight to Lisbon next week?" The first call times out — the pricing service
is momentarily overloaded. A well-built agent catches this specific failure, waits a
short interval, say one second, and retries. The retry succeeds, returns pricing data,
and the agent answers the user normally. From the user's side, nothing unusual
happened; the delay was perhaps a second longer than usual. The entire failure and
recovery happened invisibly, exactly because the agent treated a timeout as something
worth a bounded retry rather than either crashing immediately or looping on it forever.

Now contrast that with a naive agent that has no backoff and no retry limit. The first
call times out, so it retries immediately — but the service is still overloaded, so
the second call also fails, immediately, and so does the third, and the agent keeps
retrying as fast as it can, as often as it can, potentially for the full length of the
conversation. Each attempt still costs an API call and real wall-clock time, so the
user is left waiting indefinitely while the agent burns cost against a tool that was
never going to succeed on attempt four if it didn't succeed on attempt one through
three. This is the exact failure mode that a backoff schedule and a hard retry ceiling
exist to prevent.

## Backoff and ceilings, concretely

A reasonable retry policy for a transient failure like a timeout looks something like:
wait one second before the first retry, two seconds before the second, four seconds
before the third, and give up after three attempts total — doubling the wait each
time, which is what "exponential" backoff means in practice. That ceiling matters as
much as the backoff itself. Without it, a permanently broken tool — a dead endpoint, an
expired API key, a service that's been shut down — produces the same symptom as a
momentarily slow one, and the agent has no way to tell the difference except by
eventually giving up. Three attempts with growing delays between them is usually
enough to ride out a genuine transient blip while staying cheap and fast when the
failure turns out to be permanent.

## What happens after you give up

Giving up on a tool shouldn't mean crashing the agent process. The better pattern is
to treat the failure itself as information and hand it back to the model: tell it, in
effect, "the flight-pricing tool failed after three attempts," and let the model
decide what to do next within the conversation — try a different tool if one exists,
narrow the request and try again, or tell the user directly that it couldn't get
pricing right now rather than making something up. This keeps the failure contained
and visible instead of silent, which is the whole point of treating error handling as
a first-class part of tool use rather than an afterthought bolted on once something
breaks in production.
