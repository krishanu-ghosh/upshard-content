---
title: Shipping and monitoring
description: Rollout and fallbacks, provider outages and rate limits, versioning prompts and models, and watching quality after launch.
roadmap: ai-engineer
node: shipping-and-monitoring
group: production
video:
  provider: youtube
  id: "jwblSAaFnNc"
  start: 0
  end: null
creator:
  name: EuroPython Conference
  channel_url: https://www.youtube.com/@EuroPythonConference
duration_seconds: 1407
level: advanced
prerequisites: [cost-and-latency, tracing-and-observability, privacy-and-data-handling]
takeaways:
  - A model call is a dependency on someone else's capacity, so rate limits, overload and degraded latency are normal operating conditions rather than incidents.
  - Prompts and model versions are deployed artefacts and need the same versioning, review and rollback as code.
  - Quality drifts after launch without anything in your system changing, because providers update models and users change how they use the feature.
check_questions:
  - q: Why does an LLM feature need a fallback that a normal API call does not?
    a: Capacity is shared and rationed. Rate limits and overload are routine, so a feature with no second path — another model, a cached answer, a degraded mode — simply stops working during someone else's traffic spike.
  - q: How can quality fall when you have not deployed anything?
    a: The model behind the endpoint can be updated, your retrieval corpus grows and changes, and users bring new kinds of requests. All three move the system's behaviour without touching your code.
deeper:
  - label: "AI Engineer — Observability, evaluation and experimentation in production"
    url: https://www.youtube.com/watch?v=JsCCrBF7F1g
related_formulas: [littles-law]
---

## Someone else's capacity

Here's the operating fact that changes everything: the most important
component of your feature runs on hardware you don't own, shared with every
other customer of that provider at the same moment. A rate limit is not an
error condition to be surprised by. It's the ordinary way shared capacity
gets rationed across customers. An overload response can arrive because of
someone else's traffic spike that has nothing to do with your usage. Latency
genuinely varies by time of day for reasons entirely outside your system.
Designing around that reality, instead of treating each occurrence as a
one-off incident to investigate, is the single biggest mindset shift from
running a typical internal service to running one built on a third-party
model API.

## Canary and shadow deployment, adapted for non-determinism

A **canary deployment** rolls a change out to a small slice of real traffic
first and watches key metrics before expanding to everyone — standard
practice, familiar from ordinary software deployment. For an LLM feature, the
metrics you watch need to include quality signals specific to this domain,
not just the usual latency and error rate. A canary for a new prompt version
should be compared against the
[eval set](/roadmaps/ai-engineer/building-an-eval-set) score and against
real-time proxies like retry rate on that slice of traffic. Infrastructure
health signals alone would miss a purely behavioral regression entirely. A
**shadow deployment** goes a step further: it runs a new prompt or model
version in parallel against a real copy of live traffic, without ever serving
its output to a user, and compares its outputs directly against the current
production version's on the exact same inputs. That's valuable precisely
because it surfaces behavioral differences risk-free, before any user is
exposed to whatever the new version might get wrong.

## Feature flags for prompts and models

Put a specific prompt version and a specific model version behind a feature
flag instead of hard-coding them into application logic. That buys you an
instant, zero-deploy rollback the moment a regression shows up in
production — flip the flag back rather than revert and redeploy code. It also
enables targeted experimentation: run two prompt variants side by side on
different user segments and compare real outcomes directly. That's the
[A/B testing](/roadmaps/ai-engineer/why-evals-matter) half of evaluation,
applied to production traffic rather than an offline eval set.

## Fallbacks worth actually having in place

**A second provider or model**, with its prompt already tested against it
ahead of time, is the fallback teams most often skip and most often regret
skipping during an outage. It's far easier to set up calmly in advance than
to improvise under the pressure of a live incident. **A degraded mode** —
serving retrieval results alone, without a final generation step, say — is
often genuinely useful on its own, and costs almost nothing to run when the
generation model itself is unavailable. **A cached answer** for common,
repeated requests can serve something slightly stale rather than nothing at
all during an outage. **An honest failure message**, used when none of the
above applies, stating plainly that the feature is temporarily unavailable,
beats a fabricated answer produced under pressure to look like it's working
when it isn't.

## Versioning prompts and models as first-class deployed artefacts

Prompts are logic, as covered in
[prompt anatomy](/roadmaps/ai-engineer/prompt-anatomy), and a change to one
can alter a system's behavior as thoroughly as any code change can. Treat
them accordingly: held in version control, reviewed by another person before
merging, deployed with the rest of the application rather than edited live in
a provider's console, and rolled back through the same mechanism as any other
code change. Pin specific model versions wherever a provider allows it. An
endpoint that silently points at whatever the provider's latest model happens
to be is effectively an unreviewed change to your product, landing on its own
schedule — and you'll generally hear about it from an affected user before
you hear it from your own release notes. Pinning turns that silent drift into
a deliberate upgrade: you schedule it, run your full eval set against it
first, and roll it out on your own terms. Recording which prompt version and
which model version produced each trace means that when quality visibly
shifts, the first diagnostic question — what actually changed — has a direct
answer instead of needing to be reconstructed from memory after the fact.

## SLOs adapted for a genuinely non-deterministic system

Traditional service-level objectives, built around uptime percentage and
fixed latency percentiles, still apply and are still worth tracking. They
just aren't sufficient on their own for an LLM feature, because a feature can
be fully "up" by every traditional measure and still produce systematically
poor answers. A meaningful SLO set for an LLM feature adds an explicit
quality objective — eval score staying above a floor, or a user-facing proxy
like negative-feedback rate staying below a ceiling — tracked and alerted on
with the same seriousness as the infrastructure metrics. Postmortem culture
has to adapt too. "This specific run couldn't be exactly reproduced" is an
expected, routine property of a probabilistic system working as designed, not
evidence the investigation was incomplete. A useful postmortem here looks at
the trace of what actually happened in the failing run, names the category of
failure it represents, and decides what eval case or guardrail should be
added to catch that category next time — rather than chasing an exact
reproduction a non-deterministic system may never give you.

## The loop this whole roadmap has been building toward

Watching quality in production is not a separate concern bolted onto the end
of building a feature; it is the thing every other group in this roadmap has
been assembling pieces of. Ship a feature. Trace every real run, per
[tracing and observability](/roadmaps/ai-engineer/tracing-and-observability).
Read a genuine sample of the failures that show up in those traces, weekly if
at all possible, rather than relying purely on automated metrics to surface
everything that matters. Turn the patterns found in that reading into new,
concrete eval cases per
[building an eval set](/roadmaps/ai-engineer/building-an-eval-set). Fix what
the evals now catch. Ship the fix, and watch it again. That closed loop —
ship, trace, read, measure, fix, ship again — is the actual discipline behind
every lesson in this roadmap, far more than any single technique covered
along the way.
