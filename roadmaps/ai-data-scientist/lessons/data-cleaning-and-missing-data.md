---
title: Data Cleaning and Missing Data
description: Why the right fix for a missing value depends on why it's missing, plus practical strategies for duplicates and messy data.
roadmap: ai-data-scientist
node: data-cleaning-and-missing-data
group: exploratory-data-analysis
video:
  provider: youtube
  id: "bDhvCp3_lYw"
  start: 0
  end: null
creator:
  name: Alex The Analyst
  channel_url: https://www.youtube.com/@AlexTheAnalyst
duration_seconds: 2317
level: intermediate
prerequisites: [the-eda-process]
takeaways:
  - Missing data falls into three patterns (MCAR, MAR, and MNAR), and the correct cleaning strategy depends on which one you're facing, not on a single default rule like always filling with the mean.
  - Dropping rows or columns is the easiest option but the most dangerous one, because it silently throws away information and can bias your dataset if the missingness isn't random.
  - Imputation should match the variable's distribution and relationships — a skewed numeric column wants the median, not the mean, and a value that depends on other columns wants a model-based fill, not a constant.
  - Duplicate detection is harder than calling one function, because near-duplicates from formatting differences or multiple updates to the same record won't be caught by an exact-match check.
check_questions:
  - q: What do MCAR, MAR, and MNAR stand for, and why does the distinction matter?
    a: "They stand for missing completely at random, missing at random, and missing not at random, and the distinction matters because MCAR lets you drop or simply impute rows safely, while MAR and MNAR require strategies that account for what's driving the missingness or you'll bias your results."
  - q: Why might filling missing values with the mean be a bad choice?
    a: Because the mean is pulled toward extreme values in a skewed distribution, so it can sit outside where most real values fall, and because it also artificially shrinks variance and weakens any correlation that variable had with others.
  - q: Why can an exact row-match check miss real duplicates?
    a: Because the same real-world record can appear with different formatting, whitespace, or a partially updated field, so two rows that represent the same entity may not be byte-for-byte identical even though they should be treated as one.
deeper:
  - label: "What is Data Cleaning? | Data Fundamentals for Beginners"
    url: "https://www.youtube.com/watch?v=WpX2F2BS3Qc"
related_formulas: []
---

## Missing data is not one problem

Every dataset has gaps, and the instinct is to reach for one fix — drop the row, fill with zero, fill with the mean — and move on. That instinct is what causes quiet, hard-to-detect bias in a model later. The right fix depends on *why* a value is missing, and statisticians sort that reason into three categories, usually called MCAR, MAR, and MNAR. Getting this classification right, even roughly, matters more than which specific imputation technique you pick afterward.

## MCAR, MAR, and MNAR, with concrete examples

**Missing completely at random (MCAR)** means the chance a value is missing has nothing to do with any variable in your dataset, observed or not — a sensor that randomly drops a reading due to a network hiccup is a good example. Under MCAR, dropping the affected rows doesn't bias your results, because the rows you keep are still a representative sample.

**Missing at random (MAR)** means the missingness is related to *other variables you have*, just not to the missing value itself. For example, older survey respondents might be less likely to answer a question about smartphone usage — age predicts the missingness, but conditional on age, there's no further pattern. Here, dropping rows biases you toward younger respondents' answers, but you can often repair this with imputation that uses age (or other related columns) to predict the missing value.

**Missing not at random (MNAR)** is the hard case: the missingness depends on the value itself. People with very high incomes are disproportionately likely to leave an income field blank on a survey. No amount of looking at other columns fully explains that pattern, because the reason for the gap is the very thing you can't observe. MNAR usually can't be fixed with clever imputation alone — it needs either domain knowledge to model the mechanism, or an explicit acknowledgment in your writeup that the remaining data is biased in a known direction.

In practice, you rarely get a formal statistical test that proves which category you're in. You make a judgment call: look at whether missingness correlates with other columns (checking for MAR), think about the real-world process generating the data (checking for MNAR), and treat anything left over as a working MCAR assumption.

## Choosing a strategy once you know the pattern

For MCAR, dropping rows is defensible if you're only losing a small fraction of your data; dropping an entire column only makes sense if it's missing so often (commonly cited thresholds are 40-60%+) that no imputation could make it trustworthy. For MAR, imputation that conditions on related columns works well: filling a missing age with the median age *within that person's job category*, rather than the global median, preserves more of the real signal. For MNAR, consider adding a binary "was this missing" indicator column alongside your imputed value — this lets a downstream model learn that missingness itself may be informative, which is often truer to the data than pretending the problem is solved.

For numeric columns, the median is usually safer than the mean, because skewed data (income, latency, order size) has a mean that gets dragged toward its tail. For categorical columns, a dedicated "Unknown" or "Not Reported" category is often more honest than filling with the mode, especially when the missingness itself might be MAR or MNAR — a silent mode-fill makes a gap in the data pretend to be the most common real answer.

## Duplicates and messy data beyond nulls

Missing values are the most obvious form of dirty data, but duplicates and formatting inconsistencies cause just as much damage. An exact-match duplicate check (`df.duplicated()`) only catches rows that are byte-for-byte identical, which misses the far more common case: the same customer entered as "Jon Smith" and "Jon  Smith" (extra whitespace), or a record that was updated once and now exists as two slightly different snapshots. Catching these requires normalization first — trimming whitespace, lowercasing text fields, standardizing date formats — and then fuzzy matching or key-based deduplication (grouping by a stable identifier like email or customer ID rather than trusting exact row equality).

Dirty data also shows up as inconsistent units (some durations in seconds, some in minutes, mixed in one column), inconsistent category spellings ("CA", "Calif.", "California"), and placeholder values that mean "missing" without using a true null (-1, 999, or an empty string masquerading as a value). None of these show up in a `.isnull()` check, which is exactly why the univariate pass from the EDA process — looking at the actual distinct values in each column — catches problems that a purely automated missing-value scan would miss entirely. Cleaning is as much about reading your data as it is about running a function.
