---
title: Tokens per second and throughput
description: Per-request generation speed versus aggregate system throughput — two different numbers that can move in opposite directions.
roadmap: inference-engineering
node: tokens-per-second-and-throughput
group: latency-and-throughput
video:
  provider: youtube
  id: "Y2qc0UhDSnc"
  start: 0
  end: 660
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1225
level: advanced
prerequisites: [time-to-first-token]
takeaways:
  - Tokens per second per user measures how fast one request streams, while tokens per second per GPU (aggregate throughput) measures how much total work the system is getting done across every concurrent user.
  - These two numbers can move in opposite directions — batching more requests together to raise aggregate throughput typically slows down how fast any single request streams.
  - Which one actually matters depends on the product — a single interactive chat session cares about per-user speed, while a bulk offline job cares almost entirely about aggregate throughput.
check_questions:
  - q: Why can raising aggregate throughput make an individual user's experience worse?
    a: Aggregate throughput goes up by serving more requests concurrently on the same hardware, typically through larger batches. A larger batch means the GPU spends more time per step working through more requests at once, which slows down how quickly any single one of those requests receives its next token.
  - q: For which kind of product does per-user tokens-per-second matter more than aggregate throughput?
    a: An interactive product where a person is reading the response as it streams — a chat interface, a coding assistant — cares directly about per-user speed, since that is what the person waiting actually experiences.
deeper: []
related_formulas: [littles-law]
---

## Two speeds, not one

"How fast is this system" has two genuinely different answers depending on
what you are asking about. **Tokens per second per user** measures how
quickly one specific request streams its output — the number that governs
how a single person experiences the response. **Tokens per second per GPU**
(or aggregate throughput) measures the total volume of tokens the whole
system produces across every concurrent request, which is the number that
determines how many users a given amount of hardware can actually serve.

## Why they trade against each other

A GPU processing one request at a time is not using its full compute
capacity during decode, which is memory-bandwidth-bound — covered in
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound). Batching several requests together lets the GPU do more
useful work per step, raising aggregate throughput. But that same batching
means each individual request's next token takes a little longer to arrive,
because the GPU is now splitting its attention across more requests at once.
Pushing batch size up keeps raising aggregate throughput and keeps slowing
down per-user speed, until per-user latency becomes unacceptable — which is
exactly the tradeoff the [next lesson](/roadmaps/inference-engineering/latency-vs-throughput) covers in full.

## Which number to optimise for

The right number to chase depends entirely on the product. An interactive
chat interface, where a person is reading the response as it streams, lives
or dies on per-user tokens per second — a slow stream is a visibly bad
experience regardless of how much total throughput the system is achieving
elsewhere. A bulk batch job — summarising ten thousand documents overnight
with nobody watching any single one stream — cares almost entirely about
aggregate throughput, since total completion time is what matters, not the
pace of any individual request.

[Little's Law](/formulas/littles-law) connects both of these to a third
quantity worth knowing: the average number of requests a system has in
flight at once equals the arrival rate multiplied by the average time each
request spends being served — which is the concrete arithmetic behind why
raising concurrency (to improve aggregate throughput) directly raises how
many requests a server needs to be able to hold in flight simultaneously.

## A worked illustration of the tradeoff

Consider a GPU that can produce 2,000 tokens per second in total when
running at a batch size of twenty concurrent requests. Divided evenly, that
is 100 tokens per second per user — fast enough that a reader perceives the
response as streaming smoothly. Pushing the batch size to eighty concurrent
requests might raise total throughput to 3,200 tokens per second, a genuine
sixty percent gain in aggregate work done — but divided across eighty users
that is only 40 tokens per second each, noticeably slower from any single
user's point of view, even though the system as a whole is doing more. Which
of these two batch sizes is the right choice depends entirely on whether the
product in question is twenty people in an interactive conversation or
eighty requests in an overnight batch job, which is exactly why this number
cannot be optimised in the abstract — it has to be chosen against the actual
product it serves.
