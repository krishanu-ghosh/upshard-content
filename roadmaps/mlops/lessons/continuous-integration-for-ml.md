---
title: Continuous integration for ML
description: CI for ML means testing code, data, and model config on every change — not just running the old test suite against a new commit.
roadmap: mlops
node: continuous-integration-for-ml
group: ci-cd
video:
  provider: youtube
  id: "9BgIDqAzfuA"
  start: 0
  end: null
creator:
  name: DVCorg
  channel_url: https://www.youtube.com/@dvcorg8370
duration_seconds: 1065
level: intermediate
prerequisites: [model-packaging-formats]
takeaways:
  - Continuous integration for ML must trigger on three kinds of change, not one — code, data, and model/pipeline config — because any of the three can independently change what the model does.
  - A CI run for an ML project should include a cheap, fast retraining smoke test and a data-validation check, not just unit tests, because unit tests alone can't catch a schema change or a quality regression in a new dataset.
  - CI for ML needs a model-quality gate (a threshold the new model's metrics must clear) in addition to a green build, because "the pipeline ran without crashing" and "the resulting model is good enough to ship" are two different claims.
check_questions:
  - q: Why does ML CI need to trigger on data changes, not just code changes?
    a: Because the model that a pipeline produces depends on the data it was trained on just as much as on the code, so a new dataset can silently change model behavior even when no code changed.
  - q: What is a retraining smoke test, and why run one in CI instead of only unit tests?
    a: It is a fast, cut-down run of the actual training pipeline on a small sample of data to confirm the pipeline still runs end to end and produces an artifact, which unit tests on individual functions can't verify because they never exercise the full pipeline together.
  - q: Why is "the build passed" not the same as "the model is good enough to merge"?
    a: A passing build only confirms the code ran without errors, while a model-quality gate checks the actual output, such as accuracy or a fairness metric, against a required threshold, and a pipeline can run cleanly while producing a worse model.
deeper: []
related_formulas: []
---

## Three triggers, not one

Continuous integration in ordinary software has a simple contract: every
push to the repository runs a build and a test suite, and a human finds
out quickly if something broke. The trigger is code. For an ML project,
that contract is too narrow, because a model's behavior is a function of
code *and* data *and* the configuration that glues a pipeline together
(feature lists, hyperparameters, preprocessing steps). Any one of those
three can change on its own. A data engineer can update the table a
training job reads from without touching a line of Python. Someone can
bump a hyperparameter in a YAML config to try an idea. Neither action
looks like a code change to a traditional CI system, so a CI setup that
only watches the code repository will miss both — and the model that
reaches production can silently differ from the one that was last
reviewed. ML CI has to watch all three surfaces and run the same
pipeline-level checks regardless of which one moved.

## What actually needs to run

Once the trigger is right, the checks themselves need to go further than
"does the test suite pass." A normal unit test suite for an ML codebase
can confirm that a preprocessing function handles a null correctly or
that a custom loss function computes the right number on a toy input —
useful, but it never exercises the pipeline as a whole, and it says
nothing about the data itself. Two checks close that gap. The first is
data validation: before training even starts, CI should assert that the
incoming dataset matches an expected schema (right columns, right types,
values in plausible ranges) and flag statistical surprises, like a
categorical column that suddenly has a new value or a numeric column
whose distribution shifted hard. Catching that here, in minutes, beats
discovering it three hours into a full training run, or worse, after a
bad model is already serving traffic. The second is a retraining smoke
test: a deliberately small, fast version of the real training run — a
thousand rows instead of ten million, one epoch instead of fifty — whose
only job is to prove the pipeline still runs start to finish and produces
a loadable model artifact. It's not trying to produce a good model; it's
trying to prove the *pipeline* isn't broken, which a unit test suite
structurally cannot do because no individual unit test ever calls the
whole chain in order.

## The quality gate that unit tests can't give you

Even a pipeline that runs cleanly end to end can produce a model that
shouldn't ship. This is the piece with no real analogue in traditional
software CI: a model-quality gate. After the smoke test (or, on a
schedule, a full training run) produces a model, CI compares its metrics
— accuracy, F1, calibration, a fairness slice, whatever the team has
decided matters — against a required threshold, and often against the
currently-deployed model's own metrics, not just a fixed bar. If the new
model is worse, the build fails, exactly the way a build fails on a
broken unit test. This matters because "no errors were thrown" and "the
output is good" are independent claims for an ML pipeline in a way
they're much closer to equivalent for a typical web service. A
misconfigured learning rate, a leaked feature, or a subtly corrupted
label column can all produce a pipeline that completes successfully and
a model that's quietly worse than the one it would replace — and only a
metric check, not a stack trace, will ever reveal that.

## Why this discipline compounds

Skipping any one of these — triggering only on code, testing only
functions instead of the pipeline, or merging on a green build with no
quality gate — doesn't fail loudly. It fails by letting a worse model
through, which is exactly the kind of failure that doesn't throw an
error and doesn't show up until someone notices degraded predictions in
production, by which point the cause could be any of a dozen merged
changes. Getting ML CI right is what makes it possible to trust a merge
at all: every change, from whichever surface it came from, gets forced
through the same data check, the same pipeline check, and the same
quality bar before it's allowed to move forward. That trust is also the
precondition for everything the rest of this group builds on — once CI
reliably tells you a change is safe, you can start asking a harder
question: what tests should actually live inside that pipeline, which is
exactly where the next lesson picks up.
