---
title: MLOps vs. DevOps
description: What actually changes when the thing you're operating has data as a dependency, non-deterministic behavior, and failures that don't throw errors.
roadmap: mlops
node: mlops-vs-devops
group: foundations
video:
  provider: youtube
  id: "vsTImsK6z4Q"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 1045
level: beginner
prerequisites: [the-ml-lifecycle]
takeaways:
  - DevOps treats code as the sole artifact that determines behavior; ML systems add data and trained parameters as equally first-class dependencies that must be versioned and reviewed alongside code.
  - ML systems can be non-deterministic and can decay with no code change at all, so "it's deployed and the tests pass" means far less for a model than it does for ordinary software.
  - A model can be fully "up" by every infrastructure metric while being silently wrong, which means ML operations need a layer of behavioral monitoring that traditional uptime and error-rate monitoring doesn't provide.
check_questions:
  - q: Why isn't versioning code enough to reproduce an ML system, when it's usually enough for a normal application?
    a: Because a model's behavior also depends on the specific training data and resulting parameters, so the same code run against different data produces a functionally different system, which means data and model artifacts need their own version history too.
  - q: What does it mean for a model's decay to be "silent" compared to a typical software outage?
    a: The service keeps responding normally with no errors or crashes, so nothing in standard infrastructure monitoring flags a problem, even though the predictions themselves have gotten worse.
  - q: Why are ML systems harder to treat as fully deterministic than typical software?
    a: Many training processes involve randomness, such as random weight initialization or random data shuffling, so retraining on identical code and data can still produce a model with slightly different parameters and behavior each run.
deeper: []
related_formulas: []
---

## Same instinct, different object

DevOps and MLOps share a goal: ship changes safely, often, and without
relying on a person to manually babysit every release. But the thing being
operated is different in ways that aren't cosmetic. A traditional
application's behavior is fully determined by its code (and its
configuration). If you have the code at a given commit, you can, in
principle, reproduce exactly what the system does. An ML system's
behavior is determined by its code *and* the data it was trained on *and*,
often, by randomness in the training process itself. That third dependency
is what forces MLOps to diverge from DevOps practice, not as an add-on but
at the foundation.

## Data as a first-class dependency

In a normal software release, "what changed" is answered by a diff of the
code. In an ML release, the model can change even when the code doesn't —
if the training data changed. Imagine a recommendation model retrained
weekly on the last 90 days of user clicks. The training script hasn't been
touched in months, but each week's model is subtly different because the
underlying data is different: a holiday shopping spike, a new product
category, a bug upstream that started logging a field as null for two
days. None of that shows up in a code review. This is why mature ML teams
version datasets and training runs with the same rigor DevOps applies to
code — tracking exactly which data snapshot, which code commit, and which
random seed produced a given model — because "what produced this
behavior" can no longer be answered by looking at code alone.

## Non-determinism as a built-in property

Software engineers are trained to expect that running the same code on
the same input twice gives the same output; when it doesn't, that's
treated as a bug to fix. Training a model, by contrast, is often
non-deterministic by design: weights get randomly initialized, data gets
shuffled randomly into batches, some layers apply random dropout during
training. Retrain the exact same script on the exact same data twice and
you can get two models with slightly different parameters and slightly
different predictions on edge cases — without any bug being present at
all. This isn't just a testing inconvenience. It means two deployments
built from "the same" pipeline aren't guaranteed to behave identically,
so reproducibility in ML requires deliberately pinning random seeds and
recording exact dependency versions — work that has no equivalent in most
conventional software testing.

## Decay with zero code changes

The most consequential difference is what happens *after* a successful,
error-free deployment. A normal service that passes its tests and deploys
cleanly tends to keep working until something changes — new code, a
config edit, a dependency bump, increased load. An ML model can start
failing with absolutely nothing changing on the engineering side, purely
because the real world drifted away from the assumptions baked into its
training data. A credit-risk model trained on pre-2020 spending patterns
didn't get worse because of a bug; it got worse because consumer behavior
shifted. Crucially, this kind of failure produces none of the normal
alarms: no stack trace, no failed health check, no spike in 500 errors.
The service is up, fast, and wrong. That gap between "operationally
healthy" and "actually correct" is the core reason ML systems need
monitoring that watches prediction distributions and downstream outcome
metrics, not just the infrastructure signals DevOps tooling was built to
watch.

## Carrying this forward

None of this means DevOps practices are wrong for ML systems — CI/CD,
infrastructure-as-code, and automated testing are all still exactly the
right instincts. MLOps is what you get when you keep those instincts and
extend them to cover what DevOps was never designed to see: data as a
tracked dependency, non-determinism as an expected property rather than a
bug, and model quality as something that has to be actively monitored
rather than assumed to hold once it's deployed. The next lesson goes
deeper into one specific, concrete way this gap shows up — a model that
trained well but behaves differently the moment it's serving real
traffic.
