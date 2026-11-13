---
title: Exploratory data analysis
description: Looking at data before modeling it — summary statistics, distributions, correlations, and asking what the data actually represents and where it came from.
roadmap: machine-learning
node: exploratory-data-analysis
group: working-with-data
video:
  provider: youtube
  id: "JG8GRlMjp3c"
  start: 0
  end: null
creator:
  name: Krish Naik
  channel_url: https://www.youtube.com/@krishnaik06
duration_seconds: 791
level: beginner
prerequisites: [the-ml-workflow]
takeaways:
  - Exploratory data analysis is looking at data before modeling it, to understand what it represents and to catch problems before they propagate into a model.
  - Univariate analysis examines one variable at a time (its distribution, its range), bivariate examines pairs of variables (correlation, relationship), and multivariate examines several together.
  - A correlation discovered during exploration is a hypothesis to investigate, not a conclusion — association between two variables is not evidence that one causes the other.
check_questions:
  - q: Why does exploratory analysis happen before any modeling, rather than after?
    a: Because problems in the data — wrong types, impossible values, skewed distributions, missing records — are far cheaper to catch by looking directly at the data than by puzzling over a model's bad predictions later.
  - q: What is the difference between univariate and bivariate analysis?
    a: Univariate analysis looks at one variable in isolation — its distribution, its typical range, its outliers. Bivariate analysis looks at the relationship between two variables at once, such as whether one tends to rise as the other does.
deeper:
  - label: "IBM Technology — Exploratory Data Analysis"
    url: https://www.youtube.com/watch?v=QiqZliDXCCg
related_formulas: [covariance, expected-value, variance]
---

## Looking before touching

Before any model is trained, the data deserves to be looked at directly.
Exploratory data analysis is that look: summary statistics, distributions,
correlations between variables, and plain visualisation, all aimed at one
question — what does this data actually represent, and where did it come
from? Skipping this step and going straight to modeling is how a wrong
assumption about the data quietly becomes a wrong assumption baked into the
model, discovered only much later when predictions turn out strange.

## Univariate, bivariate, and multivariate

**Univariate analysis** looks at one variable at a time: its distribution
(is it roughly symmetric, or heavily skewed to one side?), its typical range,
and whether it contains outliers far from the rest of its values. This is the
simplest and most foundational pass, and it alone catches a surprising number
of data problems — a column of ages containing a value of 200, say.

**Bivariate analysis** looks at two variables together, most often to check
whether they move together — if one rises, does the other tend to rise too,
fall, or show no relationship at all? This is where a correlation between two
variables first becomes visible, long before any model is trained on them.

**Multivariate analysis** extends the same question to several variables at
once, which starts to reveal interactions a pairwise view cannot — two
variables that mean something different in combination than either does
alone.

## What a look at the data actually catches

A handful of problems surface reliably at this stage, before they ever reach
a model. A skewed distribution, most values clustered low with a long tail
of large ones, can make the mean a misleading summary and may call for a
transformation before training, a technique covered in
[feature engineering](/roadmaps/machine-learning/feature-engineering). An
unexpected spike of identical values often signals a placeholder standing in
for missing data rather than a genuine measurement — a column of ages with
suspiciously many entries at exactly zero, say. That belongs to the
[data cleaning](/roadmaps/machine-learning/data-cleaning) lesson immediately
following this one. A strong correlation between two input variables can
mean they are carrying mostly the same information, which affects how some
models, particularly linear ones, interpret their coefficients.

## The plots that do most of the work

A handful of chart types cover nearly everything this stage needs. A
**histogram** shows one variable's distribution directly — where its mass
sits, whether it is symmetric or skewed, whether it has more than one peak.
A **box plot** condenses that same distribution into five numbers (minimum,
first quartile, median, third quartile, maximum) and makes outliers visible
at a glance as points beyond the whiskers. A **scatter plot** is the
workhorse of bivariate analysis: two variables plotted against each other
reveal a linear relationship, a curved one, or no relationship at all, far
faster than scanning a table of numbers ever could. None of these require
sophisticated tooling — a few lines in any plotting library produce all
three — but skipping them in favour of jumping straight to summary
statistics is how a bimodal distribution gets mistaken for a well-behaved
one, since a single mean and standard deviation cannot tell the two apart.

## Correlation is a hypothesis, not a conclusion

Finding that two variables move together is often the most exciting moment of
an exploratory pass, and it is also where the most common mistake in this
whole field happens: treating that association as evidence that one variable
causes the other. Two variables can correlate because one genuinely causes
the other, because both are driven by some third factor neither of them
captures, or by pure coincidence in a small enough sample. Exploratory
analysis is where a correlation is noticed; it is not where it gets
explained, and the gap between those two things is worth remembering every
time a chart looks like it is telling a clean causal story.
