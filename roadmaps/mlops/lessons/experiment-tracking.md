---
title: Experiment tracking
description: Logging every run's config, metrics, and artifacts so "which setup got 94%?" has a real answer instead of a guess.
roadmap: mlops
node: experiment-tracking
group: experimentation
video:
  provider: youtube
  id: "LQvRhQwDOm0"
  start: 0
  end: null
creator:
  name: sentdex
  channel_url: https://www.youtube.com/@sentdex
duration_seconds: 1275
level: intermediate
prerequisites: [reproducibility-and-the-training-serving-gap]
takeaways:
  - An experiment tracker logs the full context of a run — hyperparameters, code version, dataset version, metrics, and artifacts — so any past result can be found, compared, and explained later.
  - Without tracking, "which run got 94%?" is answered by memory, filenames, or scrollback, all of which fail silently once run counts get into the dozens.
  - Tracking turns tuning from a sequence of one-off experiments into a searchable table, which is what makes systematic comparison and later automation possible.
check_questions:
  - q: What does an experiment tracker record beyond just the final accuracy number?
    a: The hyperparameters, code and data versions, training metrics over time, and any output artifacts for the run, not just its final score.
  - q: Why does manually naming files like model_v2_final_REAL.pt break down as a tracking strategy?
    a: Because it has no structured, queryable record of what configuration produced that file, so comparing it to other runs or reproducing it later depends entirely on someone's memory.
  - q: What underlying problem does experiment tracking solve that later MLOps practices build on?
    a: It makes a given result traceable back to the exact configuration that produced it, which is the same traceability that model registries and CI/CD pipelines for ML depend on.
deeper: []
related_formulas: []
---

## The problem before the tool

Training a model is rarely a single run. It's dozens or hundreds of runs,
each with a different learning rate, a different batch size, a different
architecture tweak, a different slice of training data. Each run produces
a number — some accuracy, some loss curve — and for a while you can hold
the top few in your head. Then you come back after a weekend, or a
teammate asks "what config got us to 94%?", and the honest answer is: you
don't know. The run that mattered is buried in a Jupyter notebook that's
since been edited, or a checkpoint file named `model_v2_final_REAL.pt`
sitting next to `model_v2_final_REAL_fixed.pt`, with no record of what
hyperparameters produced either one. This isn't a hypothetical edge case —
it's the default outcome of training without a system, and it's the
specific problem experiment tracking exists to solve.

## What a tracker actually captures

An experiment tracker (MLflow, Weights & Biases, and similar tools follow
the same core shape) wraps your training loop and logs, for every run:
the hyperparameters you set going in (learning rate, batch size, model
architecture, regularization strength); the code version, usually a git
commit hash, so you know exactly which version of your training script
produced this result; the dataset version or a reference to it, since the
same code against different data gives different results; metrics logged
over time, not just a final number, so you can see the loss curve, not
just where it ended up; and artifacts — the saved model weights, a
confusion matrix plot, a sample of predictions — attached to that specific
run. All of this gets written to a backend (a local directory, a database,
a hosted service) under a unique run ID, and a UI lets you list runs,
sort by metric, and diff two runs' configs side by side.

The mechanical trick that makes this useful isn't the logging itself —
you could print all of this to a text file — it's that it's *structured
and queryable*. "Show me every run with learning rate between 1e-4 and
1e-3 that scored above 90% validation accuracy" is a query a tracker can
answer in seconds. A folder of text files can't answer it at all without
someone manually reading through them.

## A concrete scenario

Say you're tuning a classifier and you run 40 experiments varying learning
rate, batch size, and dropout. Without tracking, you might keep a
spreadsheet you update by hand — until you forget to update it for run 23,
or copy a number wrong for run 31. With tracking, each of those 40 runs is
logged automatically the moment it starts: `mlflow.start_run()` wraps the
training call, `mlflow.log_param("learning_rate", lr)` and
`mlflow.log_metric("val_accuracy", acc, step=epoch)` run inside the loop
with no extra bookkeeping from you. Three weeks later, when someone asks
why production performance doesn't match what you reported, you don't
reconstruct the answer from memory — you open the tracker, sort 40 runs by
validation accuracy, find the best one, and read off its exact config and
the git commit it was trained from. If the saved model artifact and the
logged metrics don't match what's in production, that's a signal
something else changed later — a question you can only ask because the
original run's full context survived.

## Why this is foundational, not optional

It's tempting to treat experiment tracking as a convenience you'll add
once things get serious. In practice, the point at which you most need it
— when you have enough runs that you can no longer hold them in memory —
is also the point at which retrofitting it means losing everything that
came before. The cost of tracking from the first run is close to zero: a
few extra lines around a training loop. The cost of not tracking compounds
quietly, run after run, until a real question ("what changed between the
model that worked and the one that doesn't?") has no answer at all.

## Why this matters going forward

Experiment tracking is the record-keeping layer everything else in this
group depends on. A model registry needs to know which run produced the
model it's promoting. Hyperparameter tuning needs a structured history of
past trials to search over efficiently instead of blindly. Evaluation and
validation need to compare a candidate model's metrics against the run
that trained it, not against a number someone remembers. None of that
works if the underlying runs were never captured in the first place —
tracking is what makes every later claim about a model's provenance
checkable rather than assumed.
