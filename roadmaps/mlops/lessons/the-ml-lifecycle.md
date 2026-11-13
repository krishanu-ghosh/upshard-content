---
title: The ML lifecycle
description: The data-to-monitoring loop every ML system runs, and why MLOps treats it as one automated cycle instead of six separate hand-offs.
roadmap: mlops
node: the-ml-lifecycle
group: foundations
video:
  provider: youtube
  id: "hwtdTrBp6TA"
  start: 0
  end: null
creator:
  name: DataTalksClub
  channel_url: https://www.youtube.com/@DataTalksClub
duration_seconds: 3164
level: beginner
prerequisites: [what-is-mlops]
takeaways:
  - The ML lifecycle is a loop — data collection, training, evaluation, deployment, monitoring, and retraining — not a one-way pipeline that ends at deployment.
  - Each stage of the loop produces an input the next stage depends on, so a weak link anywhere (bad labels, no held-out evaluation set, no monitoring) silently degrades everything downstream of it.
  - MLOps is specifically the automation and instrumentation of this loop — turning each manual stage into a repeatable, observable, triggerable step instead of something a person does by hand each time.
check_questions:
  - q: Why is the ML lifecycle described as a loop rather than a straight line ending at deployment?
    a: Because a deployed model's performance is monitored, and when it degrades or new data becomes available, that feeds back into retraining, which starts the cycle again.
  - q: What goes wrong if evaluation only happens once, right after the first training run?
    a: There's no way to know when the model's accuracy on real traffic has degraded over time, since nothing is comparing current predictions against a trustworthy baseline on an ongoing basis.
  - q: What does it mean, concretely, for MLOps to "automate" a stage of the lifecycle like retraining?
    a: Instead of a person noticing degraded performance and manually rerunning scripts, a pipeline is triggered automatically — by a schedule or a monitoring alert — to pull fresh data, retrain, re-evaluate, and redeploy with minimal manual intervention.
deeper:
  - label: "MLOps Explained: End-to-End ML Lifecycle, CI/CD & Monitoring — BMR EDUCATION"
    url: https://www.youtube.com/watch?v=6kTVwX293Ec
related_formulas: []
---

## A loop, not a pipeline

It's tempting to picture building a machine learning system as a straight
line: collect data, train a model, ship it, done. That picture is wrong in
a way that matters. The moment a model is deployed, the real world keeps
moving — user behavior shifts, new categories of input appear, the
business question itself changes — while the model stays frozen exactly as
it was the day it was trained. A model that was 95% accurate at launch can
be meaningfully worse six months later without a single line of its code
changing, simply because the world it's predicting against has drifted.
That's why the ML lifecycle is drawn as a loop: data collection → training
→ evaluation → deployment → monitoring → retraining, and then back to data
collection again. Deployment isn't the finish line; it's the point where
the next lap starts.

## Walking the loop stage by stage

**Data collection** is where everything starts, and where most real-world
failure gets baked in. This isn't just "gather rows" — it means deciding
what counts as a valid label, how to handle missing values, and how to
avoid leaking information from the future into training data that
wouldn't be available at prediction time. A fraud model trained on data
that includes a "chargeback filed" field looks great in testing and fails
completely in production, because that field doesn't exist yet when a
transaction is actually happening.

**Training** takes that data and fits model parameters to it. In a mature
system this step is scripted and parameterized — the exact same training
code can be pointed at a new data snapshot and rerun without a human
rewriting anything, which is what makes retraining later in the loop
possible at all.

**Evaluation** checks the trained model against data it didn't train on,
using metrics chosen for the actual problem (accuracy alone is often
misleading — a fraud model that labels everything "not fraud" can be 99%
"accurate" while being useless). Evaluation is also where a new model
candidate gets compared against the model currently in production, so a
retrain only ships if it's actually better.

**Deployment** takes a model that passed evaluation and makes it available
to serve real predictions — behind an API, inside a batch job, embedded in
an app. This is where training/serving gaps (covered in a later lesson)
tend to surface, because the deployment environment is rarely identical to
the training environment.

**Monitoring** is the stage that's easiest to skip and most costly to
skip. It means watching not just whether the service is up, but what the
model is actually predicting: has the distribution of incoming data
shifted away from training data? Has a proxy metric for accuracy (like
user override rate, or downstream conversion) started moving the wrong
way? Without this stage, degradation is invisible until someone complains.

**Retraining** closes the loop. When monitoring flags drift, or on a fixed
schedule, or when enough new labeled data has accumulated, the cycle
starts over with fresh data feeding back into training.

## Why treating it as a manual, one-off process breaks down

Picture a team that runs this loop entirely by hand: a data scientist
pulls a fresh export every few months, retrains in a notebook, eyeballs
the accuracy number, and if it looks fine, emails the new model file to
whoever manages the deployment server. This works exactly until the person
who understands the pipeline goes on vacation, or the data export format
changes upstream and nobody notices because there's no automated check, or
three months pass without anyone retraining because nothing forced the
cadence. Each hand-off is a place where institutional knowledge that lives
in one person's head can silently break the whole chain. The failure mode
isn't dramatic — nothing crashes — it's slow decay that nobody is watching
for.

## Why this is the spine of the roadmap

Every later topic in MLOps is really about hardening one link in this
loop: making data collection and feature computation reproducible,
making deployment safe and fast, making monitoring catch the specific
kind of degradation ML systems are prone to, and making retraining
trigger automatically instead of depending on someone remembering. Holding
the whole loop in your head — not just the individual stages — is what
lets you recognize, later in this roadmap, why a gap at any single stage
threatens the model's value no matter how good the stages around it are.
