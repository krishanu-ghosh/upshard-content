---
title: Continuous delivery for models
description: Automating the path from a validated model artifact to a deployable one — CD for software, applied to models.
roadmap: mlops
node: continuous-delivery-for-models
group: ci-cd
video:
  provider: youtube
  id: "LXaLuaOUzLs"
  start: 0
  end: null
creator:
  name: The Linux Foundation
  channel_url: https://www.youtube.com/@LinuxfoundationOrg
duration_seconds: 2429
level: intermediate
prerequisites: [continuous-training]
takeaways:
  - Continuous delivery for models is the automated handoff from "this model passed its evaluation gate" to "this model is packaged, versioned, and ready to deploy" without a person manually moving files between systems.
  - The pipeline has to produce more than the model's weights — it packages the model alongside its exact dependencies, preprocessing code, and a version-pinned record of what training run and data produced it, because deployability requires reproducibility.
  - A model registry sits in the middle of CD for ML as the system of record that tracks a model's stage (staging, production, archived) and its full lineage, which is what allows a deployment step to simply ask "give me the current production model" instead of needing a human to find the right file.
check_questions:
  - q: What specific gap does continuous delivery for models close, between training and deployment?
    a: It closes the manual handoff where a validated model artifact has to be found, packaged, and moved into a deployable form by a person, replacing that with an automated pipeline that does it consistently every time.
  - q: Why does a CD pipeline for models need to package more than just the trained weights?
    a: Because weights alone aren't usable without the exact preprocessing code and dependency versions used at training time, so leaving those out risks a training-serving mismatch where the deployed model behaves differently than it did when it passed evaluation.
  - q: What role does a model registry play in CD for ML?
    a: It acts as the system of record that tracks each model version's stage and lineage, so a deployment step can retrieve "the current production model" by querying the registry instead of a person manually locating and copying the right file.
deeper:
  - label: CD4ML and the challenges of testing and quality in ML systems — Danilo Sato, via Seldon
    url: https://www.youtube.com/watch?v=adexWmMYLcw
related_formulas: []
---

## The handoff that CD removes

By the time a model has passed its evaluation gate — it cleared the
quality bar, it beat the current production model, it's the output of a
continuous training pipeline that did its job — there's still a gap
between "we have a good model" and "that model is running in production."
In a shop without continuous delivery, that gap is a person: someone pulls
the model file off wherever training left it, checks it matches the run
that was approved, figures out what dependencies it needs, builds a
serving container by hand, and pushes it somewhere. Every one of those
steps is a place for drift to creep in — a slightly different library
version in the hand-built container than what was used at training time,
a wrong file picked up because two runs finished close together, a
preprocessing script that got updated after training but before the
manual packaging step. Continuous delivery for models is the automation
of exactly that handoff: given a model that has passed its gate, produce,
deterministically and without a human in the loop, a packaged artifact
that is ready to deploy.

## What "packaged and deployable" actually requires

The core mistake to avoid here is treating a model's deployable artifact
as just its weights file. A set of weights on its own is close to
useless: a downstream service also needs the exact preprocessing logic
that turned raw input into the features the model expects, the exact
versions of the libraries the model was built with (a different version
of a tokenizer or a numeric library can silently change output), and
metadata tying the artifact back to the training run, code commit, and
dataset version that produced it. This is the training-serving
consistency problem showing up again in a new form: if the CD pipeline
packages the weights but not the matching preprocessing code, you can end
up with a model trained on one feature encoding being served behind
different preprocessing — producing predictions that look plausible and
are systematically wrong. A model-packaging format (containers, or a
standard format like the one covered earlier in this roadmap) exists
precisely to bundle all of this together as one unit so that "the
artifact that passed evaluation" and "the artifact that gets deployed"
are provably the same thing, not just probably the same thing.

## The registry as the system of record

The piece that makes this pipeline actually automatable end to end,
rather than just scripted-but-manual, is a model registry: a tracked
store of model versions, each tagged with a stage — staging, production,
archived — and its full lineage back to the run that created it.
Without a registry, "deploy the latest approved model" has no unambiguous
answer; someone has to know which of several candidate artifacts is the
right one. With a registry, promotion from "passed evaluation" to
"production" is a status change on an already-packaged artifact, and a
deployment step can be as simple as querying the registry for whatever is
currently tagged production and pulling it. This is also what makes
rollback tractable: if a newly promoted model misbehaves in production,
reverting means pointing the stage label back at the previous version,
not reconstructing it from scratch. The registry is what turns "a model
that passed its tests" into something a fully automated pipeline can
act on without a person interpreting the result.

## Why this closes the loop

Put the whole group of lessons together and the shape becomes clear: CI
catches problems in code, data, and config on every change; layered
testing makes sure a passing pipeline actually means a trustworthy model;
continuous training keeps candidates coming without waiting on a human's
schedule, gated so a worse model can't silently replace a better one; and
continuous delivery takes a gated, approved candidate and turns it into
something that can actually be deployed, consistently, every time. Skip
any one of these and the others lose most of their value — automated
training without automated, trustworthy delivery just produces validated
models that still require a risky manual step to ship. Together, they're
what makes "deploy models as often as the data demands" something a team
can do safely rather than something that only works until the one time a
manual step gets skipped.
