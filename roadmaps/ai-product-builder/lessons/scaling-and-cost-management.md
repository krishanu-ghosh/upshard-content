---
title: Scaling and Cost Management
description: Managing infrastructure and LLM API costs as usage grows, including why token-based pricing behaves differently from traditional infra bills.
roadmap: ai-product-builder
node: scaling-and-cost-management
group: shipping-and-operating
video:
  provider: youtube
  id: "FHGfSjDGUIw"
  start: 0
  end: null
creator:
  name: Theo - t3.gg
  channel_url: https://www.youtube.com/@theo
duration_seconds: 2167
level: advanced
prerequisites:
  - monitoring-and-iterating-after-launch
takeaways:
  - Traditional infrastructure costs (compute, database, bandwidth) scale roughly with user count and usage volume, and are fairly predictable once you've operated a system for a while.
  - LLM API costs scale differently — they're metered per token, and the cost of a single request can vary by an order of magnitude depending on which model and how much context you use, independent of how many users you have.
  - A feature that calls a more capable (and more expensive) model for every request can look cheap in testing and become a real line item the moment real usage volume shows up.
  - Cost isn't a one-time setup decision for an AI feature — model pricing, your own usage patterns, and the cheapest model that's "good enough" for a given task all shift over time and need periodic re-checking.
  - Watching cost per request alongside cost per user reveals problems a pure revenue-per-user view hides, especially for AI features where a small fraction of heavy users can dominate the bill.
check_questions:
  - q: How does the way LLM API costs scale differ from how traditional infrastructure costs scale?
    a: Traditional infra costs scale fairly predictably with user count and request volume. LLM costs scale with tokens processed per request, so the same feature can cost very different amounts depending on the model chosen and how much context is sent, independent of user count.
  - q: Why can an AI feature that looked cheap during development become expensive in production?
    a: Testing usually involves light, occasional usage with short inputs. Real users at volume, with longer conversations or larger context, multiply the per-request token cost by far more requests than were ever run during development, and the bill scales with that, not with the original test estimate.
  - q: Besides LLM token costs, what other costs still need active management as an app scales?
    a: Database costs (storage and query volume), compute/hosting costs (as traffic and background jobs grow), and bandwidth (serving more data to more users) — all of which still require periodic attention as usage grows, separate from anything AI-specific.
deeper:
  - label: "I thought AI was gonna get cheaper?"
    url: https://www.youtube.com/watch?v=vRS2leJLAI0
related_formulas: []
---

## Cost as an ongoing job, not a setup step

It's tempting to treat cost management as something you configure once at launch and revisit only if something goes wrong. That works reasonably well for traditional infrastructure, which tends to scale predictably — more users means more database load and more bandwidth, in a relationship you can estimate and budget for. It works much less well once LLM API calls are part of your product, because that cost behaves on a different curve, and treating it as a solved problem after the first integration is how teams get an unpleasant surprise on a bill weeks or months later.

## Why token-based pricing doesn't behave like a normal infra cost

An LLM API charges per token processed, both the tokens you send in and the tokens it generates back. That means the cost of a single request depends on three things you control directly: which model you call, how much context you include in the prompt, and how long the response is. A more capable model can cost an order of magnitude more per token than a lighter one, and a feature that sends a large document as context on every call multiplies that cost by however many times a user triggers it. None of this correlates cleanly with user count the way a database query does — two users can generate wildly different costs depending on how they use an AI feature, which makes LLM spend much harder to estimate from traffic alone.

## The trap of testing-cheap, production-expensive

During development, you call the API occasionally, with short test inputs, and the cost is trivially small — easy to overlook entirely. Once real users are hitting that feature at volume, with longer inputs and more frequent use than your testing ever exercised, the same per-request cost gets multiplied by a request count you didn't plan around. This is the specific failure mode worth watching for: a feature that was free to ignore in development can become one of the largest line items in production, and the only way to catch that before it's a crisis is to track cost per request in production from the start, not just overall revenue or user growth.

## Choosing the cheapest model that's actually good enough

Not every feature needs your most capable (and most expensive) model. A classification task, a short summarization, or a simple extraction often performs just as well on a smaller, cheaper model as on a flagship one, and the difference in cost per request can be substantial at volume. This isn't a decision to make once — as new models ship and prices shift, the cheapest model that still clears your quality bar for a given task can change, which means it's worth periodically re-testing cheaper options against your actual use case rather than assuming the model you picked at launch is still the right trade-off.

## The non-AI side of the bill still needs attention

LLM costs get the attention because they're newer and less intuitive, but database, compute, and bandwidth costs don't stop mattering just because an AI feature is also in the picture. A growing user base means more rows, more queries, more concurrent connections, and more data served — all of which scale in the more traditional, more predictable way, but still require real monitoring, indexing, caching, and occasionally a bigger instance or a different pricing tier as usage grows. Treating "scaling and cost" as synonymous with "AI cost" misses half the picture; a complete view tracks both curves, because a database that silently becomes the bottleneck is just as capable of taking a product down as an unexpectedly large LLM bill.
