---
title: Deployment strategies
description: Canary, blue-green, and shadow deployments each limit a different kind of risk when rolling out a new model version — not interchangeable safety nets.
roadmap: mlops
node: deployment-strategies
group: serving-deployment
video:
  provider: youtube
  id: "3b-WkL1Zn5s"
  start: 0
  end: null
creator:
  name: RoboSathi
  channel_url: https://www.youtube.com/@RoboSathi
duration_seconds: 601
level: intermediate
prerequisites: [model-serving-patterns]
takeaways:
  - Canary, blue-green, and shadow deployments all avoid a risky all-at-once cutover, but they protect against different failure modes and expose users to different amounts of risk.
  - Shadow deployment is the only one of the three that produces zero user-facing risk, because the new model's predictions are never actually returned to anyone.
  - Blue-green gives you an instant, clean rollback by keeping two full environments running, at the cost of running double the infrastructure during the switch.
check_questions:
  - q: Why is shadow deployment lower-risk than canary deployment?
    a: Because in a shadow deployment the new model's predictions are logged and compared but never returned to real users, so a bad model can't affect anyone, whereas a canary deployment does serve some real users the new model's live output.
  - q: What's the main operational cost of blue-green deployment compared to canary?
    a: It requires running two full, identically-scaled production environments at once so traffic can be switched or rolled back instantly, roughly doubling infrastructure cost during the transition.
  - q: Why can't shadow deployment alone validate a model that depends on user reaction, like a recommendation ranking?
    a: Because shadow mode only compares predictions offline against a baseline or ground truth — it never puts the new model's output in front of a real user, so it can't measure how users actually respond to it.
deeper:
  - label: "Deployment Strategies Explained: Blue-Green vs. Canary vs. Rolling — CodeLucky"
    url: "https://www.youtube.com/watch?v=H5z70EBtEow"
related_formulas: []
---

## The problem with flipping a switch

Once a serving pattern is chosen, the next hazard is how you actually
replace the model currently running in production with a new one. The
naive approach — stop the old model, start the new one, done — means the
very first signal that something's wrong is a flood of bad predictions or
errors hitting every single user simultaneously, with no graceful way back
except reversing the same risky, all-at-once switch. Deployment strategies
exist to replace that with something that fails small, fails visibly, and
fails reversibly. Canary, blue-green, and shadow deployment are the three
standard answers, and they're not three flavors of the same idea — each
limits a different kind of risk, and picking the right one depends on what
you're actually worried could go wrong.

## Canary: expose a slice, watch it, expand

A canary deployment routes a small percentage of real traffic — maybe 1%,
maybe 5% — to the new model version while the rest keeps hitting the
existing one. You watch the canary's metrics (error rates, latency,
prediction distributions, business outcomes if you can measure them fast
enough) against the baseline, and if they hold up, you gradually increase
the new version's share until it's serving everyone. If they don't, you
route traffic back to the old version, and only a small slice of users
were ever affected.

The real value of canary is that it tests the model against *actual*
production traffic and the full serving stack — load balancers, feature
pipelines, downstream services — in a way that no offline evaluation can
fully replicate, while bounding the blast radius of a bad model to a
fraction of users for a limited time. The cost is that it does expose real
users to the new version, and you need solid, fast monitoring to catch a
regression before the canary's traffic share grows large enough to matter.
A canary deployment with no real-time monitoring behind it is barely safer
than a hard cutover — it just makes the hard cutover happen in slow motion.

## Blue-green: two full environments, one instant switch

Blue-green deployment keeps two complete, independently scaled production
environments: "blue" (currently live) and "green" (the new version,
deployed and warmed up but not yet receiving traffic). Once green passes
its checks, traffic is switched over — often all at once — and blue is
kept running, idle, as an immediate rollback target. If green misbehaves,
traffic flips back to blue in roughly the time it takes to update a
router or load balancer, with no need to redeploy anything.

This buys something canary can't: a rollback that is instant and clean,
because the old environment was never torn down. The tradeoff is cost and
resource duplication — you're paying for two full-scale environments
during the transition — and because the switch is typically all-or-nothing,
blue-green doesn't give you the same gradual, traffic-percentage-based
signal that a canary does. It's a strong fit when you need a guaranteed
fast rollback path and can tolerate the extra infrastructure spend, but it
doesn't by itself limit how many users see the new model while it's live.

## Shadow: test with zero user-facing risk

A shadow deployment sends real production traffic to the new model
*in parallel* with the live model, but never returns the new model's
predictions to the user — they're only logged and compared against the
current model's output or against ground truth collected later. Because
no user ever sees the shadow model's predictions, there is no risk to the
user experience at all, regardless of how badly the new model performs.
This makes shadowing the right first step for a model you don't trust yet,
before exposing it through canary or blue-green.

Shadowing's limit is exactly what makes it safe: it can only validate
things you can evaluate offline, by comparing logged predictions. It
can't tell you how users actually behave differently when they're served
the new model's ranking, recommendation, or response — for that, you
eventually need live traffic, which is where canary and blue-green take
over.

## Choosing, and what comes next

These aren't mutually exclusive — a common path is shadow first to catch
gross errors with zero risk, then canary to validate against real user
behavior with bounded exposure, then either full rollout or a blue-green
switch if you need an instant rollback guarantee for the final cutover.
The next lesson in this group picks up from here: once a new version is
safely live, how do you keep it running under real production load.
