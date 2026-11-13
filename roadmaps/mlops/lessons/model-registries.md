---
title: Model registries
description: Why "what did we train" and "what is allowed to run in production" need to be two different, tracked lists.
roadmap: mlops
node: model-registries
group: experimentation
video:
  provider: youtube
  id: "iIiPo4qv97o"
  start: 0
  end: null
creator:
  name: Karndeep Singh
  channel_url: https://www.youtube.com/@karndeepsingh
duration_seconds: 1237
level: intermediate
prerequisites: [experiment-tracking]
takeaways:
  - A model registry is a versioned catalog of trained models with metadata and a promotion state (such as staging or production), separate from both a code repository and an experiment tracker.
  - Experiment tracking records what was tried; a registry records what is actually approved and allowed to be deployed, which is a smaller, curated subset.
  - Promotion through the registry gives a system a single place to answer "what's live right now, and can we roll it back" without digging through deployment scripts or chat history.
check_questions:
  - q: How does a model registry differ from an experiment tracker?
    a: An experiment tracker logs every run that was tried, while a registry holds only the curated, versioned models that have been approved to move toward or into production.
  - q: What does "promoting" a model version in a registry actually change?
    a: It changes that version's stage label, such as from staging to production, which is what downstream deployment systems read to decide which model to actually serve.
  - q: Why isn't a code repository enough to answer "what model is running in production right now"?
    a: Because code repositories version source code, not trained model artifacts and their binary weights, so they can't tell you which specific trained artifact, with which weights, is currently deployed.
deeper:
  - label: "MLFlow Tutorial | ML Ops Tutorial — codebasics"
    url: https://www.youtube.com/watch?v=6ngxBkx05Fs
related_formulas: []
---

## Trained isn't the same as approved

An experiment tracker will happily hold a thousand runs, and maybe a dozen
of them are good enough to matter. That raises a different question than
"which run scored highest": which model is actually *approved* to be
deployed, and which one is *currently* serving production traffic? Those
aren't the same question, and conflating them is where teams get into
trouble — someone deploys "the model from the shared drive" that turns out
to be an earlier, worse checkpoint than the one everyone thought was live,
because there was no single, authoritative place recording which version
was which. A model registry exists to answer exactly this: it's the
curated, versioned catalog of models that have cleared some bar, with
explicit state about where each version stands in the path to production.

## What a registry actually stores

A registry entry isn't just a file. For each registered model name, it
tracks a sequence of versions, and for each version: a pointer to the
actual model artifact (often stored elsewhere, in object storage, with
the registry holding the reference); metadata like the training run that
produced it, the dataset version, and evaluation metrics; a human-readable
description and tags; and — the part that does the real work — a stage
label such as `None`, `Staging`, `Production`, or `Archived`. Moving a
model from `Staging` to `Production` is called promotion, and it's
typically a deliberate action, not automatic: someone (or some automated
gate) decides this version has cleared validation and is ready to take
over from whatever's currently serving. Critically, the registry also
keeps the history — version 7 can be `Production` today, but version 6 is
still there, labeled `Archived`, ready to be promoted back if version 7
turns out to have a problem nobody caught in testing.

## Why this needs to be separate from a code repo

It's tempting to think "we already version everything in git, isn't that
enough?" Git versions source code and configuration well, but a trained
model is a large binary artifact (megabytes to gigabytes of weights) that
doesn't belong in a code repository's history, and more importantly,
knowing which *code* produced a model doesn't tell you which *trained
artifact* is currently deployed. Two runs of the identical training code
against the identical data can still produce different weights, because
of random initialization, data shuffling order, or hardware-level
nondeterminism. A registry tracks the artifact itself, with its lineage
back to the run and code that produced it — which is the thing a
deployment system actually needs to fetch and serve.

## A concrete scenario

Imagine a fraud-detection model goes through weekly retraining. Each
week's run gets logged by the experiment tracker along with a dozen
tuning variants. The best one gets registered as a new version of the
`fraud-classifier` model — version 14, say — and sits in `Staging` while
it runs through a validation gate (covered in the evaluation lesson
later in this group): backtesting against last week's transactions,
checking it doesn't regress on a protected slice of cases. Once it
passes, someone promotes version 14 to `Production`. The serving system
is configured to always pull "whatever is currently in `Production`" for
`fraud-classifier`, so this one action — a stage change in the registry —
is what actually ships the new model, with no redeploy of the serving
code itself. Three days later, fraud analysts notice a spike in false
positives. Instead of a scramble to figure out what changed, someone opens
the registry, sees version 14 replaced version 13 on that date, and
demotes 14 back while promoting 13 to `Production` again — a rollback
that takes seconds because the previous good version was never deleted,
just relabeled.

## Why this matters going forward

A registry is the hinge between "training produced something" and
"something is running." Hyperparameter tuning, the next lesson in this
group, searches for candidates that are worth registering in the first
place. Model evaluation and validation, two lessons ahead, defines the
gate a version has to clear before it's allowed to be promoted. And every
deployment pattern covered later in this roadmap — canary releases, blue/
green deployment, rollback — assumes there's a single, trustworthy source
of truth for "what's approved and what's live" to act on. Without a
registry, each of those later practices has to invent its own ad hoc
answer to a question the registry is built to settle once.
