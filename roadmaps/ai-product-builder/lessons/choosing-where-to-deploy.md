---
title: Choosing Where to Deploy
description: A framework for picking between serverless platforms, PaaS, and raw cloud based on traffic, team size, and how much infrastructure you want to manage.
roadmap: ai-product-builder
node: choosing-where-to-deploy
group: shipping-and-operating
video:
  provider: youtube
  id: "yfxDdQo2cyI"
  start: 0
  end: null
creator:
  name: Theo - t3.gg
  channel_url: https://www.youtube.com/@theo
duration_seconds: 2233
level: intermediate
prerequisites:
  - continuous-integration-basics
takeaways:
  - Serverless platforms like Vercel and Cloudflare handle scaling and infrastructure for you in exchange for constraints on runtime and execution time, which is the right trade for most small apps with bursty or unpredictable traffic.
  - A PaaS like Render or Railway sits a notch closer to raw infrastructure — you still think in terms of a long-running server and a database — but it still hides the provisioning, networking, and patching that raw cloud exposes.
  - Raw cloud (AWS, Azure, GCP) gives you full control over every layer, which is valuable at a certain scale or for specific compliance needs, but it also makes you responsible for every layer, including ones you didn't know existed yet.
  - "The right choice depends less on which platform is \"best\" and more on three questions: how predictable is your traffic, how many people are around to operate the infrastructure, and how much of that operational burden do you actually want to own."
  - Specific pricing numbers go stale fast; treat any dollar figure from a video or article as a rough shape of the trade-off, and check the platform's current pricing page before committing to it.
check_questions:
  - q: What's the core trade-off a serverless platform like Vercel or Cloudflare makes on your behalf?
    a: It handles scaling, provisioning, and infrastructure management automatically, in exchange for constraints like execution time limits and a particular way of structuring your backend code (e.g., as functions rather than a long-running process).
  - q: How does a PaaS like Render or Railway differ from both serverless and raw cloud?
    a: It still gives you a mental model closer to a traditional server and database that you provision and configure, but it hides the lower-level infrastructure work (networking, patching, hardware) that raw cloud leaves entirely in your hands.
  - q: What three factors should actually drive a deployment choice, according to the framework in this lesson?
    a: Your traffic pattern (steady vs. bursty/unpredictable), your team size (who's around to operate infrastructure), and how much operational burden you genuinely want to take on versus hand off to the platform.
deeper: []
related_formulas: []
---

## The real question isn't "which platform is best"

Every deployment platform comparison eventually turns into a list of logos, and that's the wrong frame. The better question is what you're optimizing for: how predictable your traffic is, how many people you have to operate infrastructure, and how much of that operational work you actually want on your plate versus handed to someone else for a fee. Two apps with identical feature sets can have completely different right answers here, because the deciding factors are about your situation, not the app's code.

## Serverless: scaling you don't think about

Platforms like Vercel and Cloudflare run your code as functions that spin up on demand and scale to zero when nobody's using them. You don't provision servers, you don't plan for peak load, and you don't get paged because traffic spiked at 2 a.m. — the platform absorbs that. The trade is a more constrained execution model: functions that run for a bounded time, specific patterns for how your backend code needs to be structured, and less control over the exact machine your code runs on. For a small-to-medium app with bursty or unpredictable traffic — which describes most early-stage products — that trade is usually a good one. You're giving up control you probably weren't going to use anyway.

## PaaS: a server, minus the parts you'd rather not manage

Render, Railway, and similar platforms sit a step closer to traditional infrastructure. You're thinking in terms of a persistent server process and a provisioned database, which fits more naturally if your app doesn't map cleanly onto short-lived functions — a long-running WebSocket connection, a background worker, a service that needs to hold state in memory. What a PaaS still takes off your hands is the layer below that: you're not racking servers, configuring load balancers, or patching an operating system. It's a middle ground, and a reasonable default when serverless's constraints don't fit your app's shape.

## Raw cloud: total control, total responsibility

AWS, Azure, and GCP give you every knob there is — which also means every knob is now something you or your team has to know about. This makes sense once you have genuine scale, specific compliance or data-residency requirements, or infrastructure needs specific enough that a managed platform's defaults get in your way. For a small team building and shipping a product, raw cloud is usually more infrastructure than the problem calls for; the control it offers is only valuable if you have the operational capacity to use it responsibly, and most early-stage teams don't yet.

## Matching the platform to the team, not the hype

The honest framework is: small team, unpredictable traffic, no dedicated infrastructure person → lean serverless. Need a persistent process or more backend flexibility but still don't want to run infrastructure → PaaS. Genuine scale, compliance needs, or a team that includes people whose job is operating infrastructure → raw cloud becomes worth its cost. None of these are permanent choices — plenty of products start on a serverless platform and migrate to raw cloud years later once their needs actually justify it, not before.

## Prices change; check before you commit

Any specific dollar figure you see in a deployment comparison — free tier limits, per-request pricing, bandwidth costs — is a snapshot from whenever that content was made, and platforms adjust pricing often enough that those numbers shouldn't be treated as current. Use comparisons for the shape of the trade-off (serverless is usage-based, a PaaS often has a flatter base cost, raw cloud bills for everything separately), and pull up the platform's actual current pricing page before making a decision that depends on exact numbers.
