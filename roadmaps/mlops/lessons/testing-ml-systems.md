---
title: Testing ML systems
description: Accuracy isn't a test suite — ML systems need data tests, training-code tests, and behavioral tests on specific known cases.
roadmap: mlops
node: testing-ml-systems
group: ci-cd
video:
  provider: youtube
  id: "hhMUE0_vAs4"
  start: 0
  end: null
creator:
  name: Jesper Dramsch – Non-hype Machine Learning
  channel_url: https://www.youtube.com/@JesperDramsch
duration_seconds: 1101
level: intermediate
prerequisites: [continuous-integration-for-ml]
takeaways:
  - A single aggregate accuracy number can hide serious, specific failures, because a model can be right on average while being reliably wrong on an identifiable subgroup or edge case.
  - ML systems need three distinct layers of tests — data validation, training-code unit tests, and model-behavior tests — and each catches a failure the other two structurally cannot.
  - Behavioral tests should be written against known, specific cases (a negation, a fixed input, a protected-attribute swap) the same way software tests are written against known edge cases, rather than relying on a drop in aggregate accuracy to notice.
check_questions:
  - q: Why is overall accuracy not sufficient as a test suite for a model?
    a: Because a single aggregate number can stay high while the model fails consistently on a specific subgroup or input pattern that the average simply dilutes away.
  - q: What is the difference between a training-code unit test and a model-behavior test?
    a: A training-code unit test checks that a specific function in the pipeline, like a feature transform, computes the correct output on a known input, while a model-behavior test checks the trained model's predictions on specific inputs to see if it behaves sensibly, including on cases never in the test or training data.
  - q: Why do data validation tests need to run before training rather than only checking the trained model afterward?
    a: Because a model trained on bad data will usually still train "successfully," so by the time you check the model's output the corrupted input has already been baked into every weight, and catching it before training is far cheaper than catching it after.
deeper:
  - label: Behavioral Testing of ML Models (Unit tests for machine learning) — Jay Alammar
    url: https://www.youtube.com/watch?v=Cse-3MM7mso
related_formulas: []
---

## One number is not a test suite

A software test suite is a list of specific claims: given this input,
expect that output. Accuracy, by contrast, is a single number averaged
over an entire dataset, and averages are excellent at hiding exactly the
kind of failure you most need to catch. A spam classifier can sit at 97%
accuracy while reliably misclassifying every email that mentions a
competitor's product name, because that pattern shows up in a small
enough slice of the test set to barely move the aggregate. A resume
screener can look fine in overall accuracy while systematically scoring
one demographic lower, again invisible in the top-line number. Treating
"accuracy is high" as equivalent to "the model is tested" throws away the
one thing that makes testing useful in traditional software: the ability
to assert something specific and know immediately when it breaks. ML
systems need that same specificity, applied at three different layers,
because each layer fails in a different way and for a different reason.

## Layer one: does the data look like what we expect

The earliest and cheapest place to catch a problem is before training
even starts. Data validation tests assert things about the input: this
column should never be null, this categorical field should only take
these five values, this numeric field should fall in this range, the
label distribution shouldn't have shifted by more than X from last
week's. None of this checks the model — it checks the ingredients. The
reason this layer has to exist is that a corrupted or drifted dataset
usually doesn't cause training to fail; it causes training to *succeed*
on bad data, producing a model that learned the wrong thing with total
confidence. By the time accuracy on a held-out set looks off, the bad
data is already baked into every weight, and diagnosing "which of the
thousand recent changes caused this" is far harder than it would have
been to catch at ingestion.

## Layer two: does the training code do what it's supposed to

This layer looks the most like conventional software testing, because it
is: ordinary unit tests against the deterministic parts of a training
pipeline. Does the feature-engineering function turn a raw timestamp into
the right day-of-week encoding? Does the class-balancing step actually
produce balanced batches? Does a custom loss function return the expected
value on a hand-computed example? These are fast, isolated, and exactly
as valuable as unit tests always are — they catch a broken transform
before it ever touches real data, and they're the layer most teams
already know how to write, because it requires no ML-specific tooling at
all, just ordinary testing discipline applied to the pipeline's
functions.

## Layer three: does the trained model behave sanely

The layer most teams skip is testing the *model itself*, not just the
code that produced it, against specific known cases — the ML equivalent
of a unit test's "given this input, expect that output." A sentiment
model should flip its prediction when you negate a sentence ("great
service" vs. "not great service"); if it doesn't, something about how it
weights negation is wrong, and no aggregate accuracy score reliably
surfaces that single failure mode. A credit model should behave
consistently when you swap a protected attribute like a name associated
with a different demographic while holding everything else fixed; if the
prediction moves, that's a specific, testable fact about the model worth
catching before deployment, not after a complaint. These behavioral
tests don't replace a held-out evaluation set — they complement it by
asserting invariants (the model should behave this way under this
transformation) and performance on hand-picked, known-hard cases that a
random sample might never happen to include.

## Why all three together

Each layer catches a failure mode invisible to the other two: bad data
can produce a model that trains cleanly and scores fine on data drawn
from the same bad distribution; broken training code can compute a
correct-looking loss while silently never updating the right weights;
and a model can have excellent aggregate accuracy while failing hard on
a specific, predictable case nobody wrote a test for. Treating any one
layer as sufficient is how teams end up surprised by a production failure
that, in hindsight, a five-line assertion would have caught during
review. Building this layered discipline is also what makes the next
step possible: once you can trust that a model has actually passed real
tests rather than just "the pipeline ran," you can start letting
retraining happen automatically instead of only by hand, which is exactly
the subject of continuous training.
