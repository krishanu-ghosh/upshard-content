---
title: Data cleaning
description: Missing values, duplicates, outliers, and inconsistent formatting — finding and handling these, and the judgment calls involved.
roadmap: machine-learning
node: data-cleaning
group: working-with-data
video:
  provider: youtube
  id: "FPvMBl8LvGA"
  start: 0
  end: null
creator:
  name: Emma Ding
  channel_url: https://www.youtube.com/@emma_ding
duration_seconds: 326
level: beginner
prerequisites: [exploratory-data-analysis]
takeaways:
  - Missing data can be dropped, imputed with a reasonable estimate, or flagged with an indicator column — and which is right depends on why the data is missing, not just how much of it is missing.
  - Dropping rows with missing values is only safe when missingness is unrelated to anything that matters for the problem; otherwise dropping systematically removes a specific kind of case rather than a random sample.
  - Duplicates, inconsistent category labels and wrong data types are quieter problems than missing values, but each one silently corrupts a model's training exactly as effectively.
check_questions:
  - q: Why is "just drop the rows with missing values" sometimes the wrong answer even when there are few of them?
    a: Because if missingness is related to the thing you are trying to predict — people who skip an income question tend to have unusually high or low incomes, say — dropping those rows removes a specific kind of case rather than a random slice, biasing what remains.
  - q: What does an indicator column for missingness let a model learn that imputing alone does not?
    a: Imputing a plausible value hides the fact that a value was ever missing. An indicator column preserves that information, which matters whenever the fact that something was missing is itself predictive.
deeper:
  - label: "Grow with Google — Intro to Data Cleaning"
    url: https://www.youtube.com/watch?v=0l_SlOq34Ug
  - label: "Alex The Analyst — What is Data Cleaning?"
    url: https://www.youtube.com/watch?v=WpX2F2BS3Qc
related_formulas: []
---

## Why this is judgment, not a checklist

Data cleaning sounds mechanical and is not. Every real dataset arrives with
problems — missing values, duplicated records, outliers, inconsistent
category labels, numbers stored as text — and for each one there is a real
decision to make rather than a single correct procedure to apply. Getting
this stage wrong is one of the most common ways a project fails quietly: the
model trains without error, produces plausible-looking predictions, and is
simply wrong in ways nobody traces back to a cleaning decision made weeks
earlier.

## Missing values: the central judgment call

When a value is missing, three options exist, and the right one depends on
*why* it is missing, not just how often.

**Drop it.** Removing rows with missing values is simplest, and it is safe
only when missingness is essentially random with respect to the problem. If
people who skip a sensitive survey question tend to have unusually high or
low values for exactly the thing the question asked about, dropping those
rows does not produce a smaller random sample — it produces a biased one,
systematically missing a specific kind of case.

**Impute it.** Filling in a plausible value — the column's average, its most
common value, or a value predicted from the other columns — keeps every row,
at the cost of inventing data that was never actually observed. This is
reasonable when missingness looks essentially random and the imputed value is
unlikely to meaningfully mislead the model.

**Flag it.** Adding a separate indicator column recording that a value was
missing, alongside an imputed value filling the gap, preserves information
that straightforward imputation discards. This matters whenever the fact of
something being missing is itself predictive — a customer who never answers
an income question may differ systematically from one who does, regardless
of what number eventually gets imputed in that field.

## The quieter problems

Missing values get the most attention because they are the most visible.
Several other problems are just as corrosive and considerably easier to
overlook.

**Duplicate rows** inflate the apparent importance of whatever they
duplicate, and a model can end up effectively over-weighting one case simply
because it appears in the data three times rather than once.

**Inconsistent category labels** — `"NY"`, `"New York"` and `"ny"` recorded
as though they were three different categories — silently fragment what
should be a single group, diluting the signal a model could otherwise extract
from it.

**Wrong data types** — a date stored as plain text, a numeric column read in
as a string because one row contains a stray comma — can cause a value to be
treated as categorical when it is really continuous, or sorted alphabetically
when it should be sorted numerically, producing a model that has quietly
learned the wrong kind of relationship entirely.

None of these announce themselves the way a clearly missing value does.
Catching them is a large part of what the
[exploratory analysis](/roadmaps/machine-learning/exploratory-data-analysis)
immediately preceding this stage is actually for.
