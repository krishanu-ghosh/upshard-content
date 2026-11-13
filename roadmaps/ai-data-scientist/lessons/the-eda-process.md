---
title: The Exploratory Data Analysis Process
description: A systematic order for looking at a new dataset before modeling — shape and types, distributions, relationships, then outliers.
roadmap: ai-data-scientist
node: the-eda-process
group: exploratory-data-analysis
video:
  provider: youtube
  id: "xi0vhXFPegw"
  start: 0
  end: null
creator:
  name: Rob Mulla
  channel_url: https://www.youtube.com/@robmulla
duration_seconds: 2421
level: beginner
prerequisites: [notebooks-and-reproducible-analysis, time-series-fundamentals]
takeaways:
  - Exploratory data analysis follows a rough order — shape and types first, then single-variable distributions, then relationships between variables, then outliers — because each step tells you what questions the next step should even ask.
  - A dataset's shape and dtypes catch the cheapest, most damaging mistakes early, like a numeric column silently loaded as text or a date column that's actually a string.
  - Looking at one variable at a time before comparing variables keeps you from misreading a correlation that's really just two skewed distributions colliding.
  - Outliers are not automatically errors — the right move is to classify each one as a data error, a rare-but-real event, or a sign your model needs a different approach, not to delete it on sight.
check_questions:
  - q: Why check shape and dtypes before looking at distributions?
    a: Because dtype and shape problems, like a date stored as a string or a column with fewer non-null values than expected, invalidate anything you compute afterward, so catching them first saves you from re-doing every later step.
  - q: Why look at single variables before looking at relationships between variables?
    a: "Because a relationship you see in a scatterplot or correlation matrix can be an artifact of each variable's own skew or outliers rather than a real connection, so you need to understand each variable alone before trusting how they move together."
  - q: Is an outlier always a mistake to remove?
    a: No — an outlier can be a data entry error, a genuinely rare event, or a sign that a single model won't fit the whole population well, and each of those three calls for a different response, not automatic deletion.
deeper:
  - label: "Exploratory Data Analysis in Pandas"
    url: "https://www.youtube.com/watch?v=Liv6eeb1VfE"
related_formulas: []
---

## Why order matters before you model anything

Exploratory data analysis, or EDA, is the work you do between "I have a dataset" and "I trust this dataset enough to model it." It's tempting to skip straight to building a model, especially when a dataset looks clean in a quick preview. But a model trained on data you haven't actually examined will happily learn from typos, duplicate rows, and a corrupted sensor reading as if they were real signal. EDA is not one technique — it's a sequence of passes, each one narrower than the last, where what you find at each stage determines what you look for next. Doing the passes out of order wastes time: there's no point building a correlation heatmap before you've confirmed your numeric columns are actually numeric.

## Step 1: Shape and types before anything else

Start by asking three boring questions: how many rows and columns, what type is each column, and how much of each column is missing. In pandas this is `df.shape`, `df.dtypes`, and `df.isnull().sum()` — three lines of code, but they catch an outsized share of real problems. A `price` column loaded as `object` instead of `float64` usually means a stray currency symbol or a comma thousands-separator snuck into the data and silently blocked numeric parsing. A `date_joined` column that's still a string means every later time-based computation is wrong until you explicitly parse it. Row count matters too: if you expected 50,000 customer records and got 48,200, somewhere a join dropped rows, and you want to know that before you compute anything that assumes completeness.

## Step 2: One variable at a time

Once types are sane, look at each variable in isolation before comparing any two. For numeric columns, that means the five-number summary (min, quartiles, max) plus a histogram — not just the mean, because the mean of a skewed distribution (like household income or session duration) can sit far from where most of your data actually lives. For categorical columns, it means a frequency count: does one category dominate 95% of rows, making it nearly useless as a predictor? Are there near-duplicate categories like "NY", "N.Y.", and "New York" that should be one value? This univariate pass is also where you first notice class imbalance in a target variable — a fraud-detection dataset that's 99.8% "not fraud" tells you immediately that plain accuracy will be a useless evaluation metric later.

## Step 3: How variables move together

Only after you understand each variable alone should you start comparing pairs. A correlation matrix or a grid of scatterplots is the usual next step for numeric variables, and a grouped bar chart or box plot for a numeric variable against a category. The reason to wait is that bivariate patterns are easy to misread when you don't already know each variable's own shape. Two variables that are each heavily skewed by the same handful of extreme rows can show a strong-looking correlation that has nothing to do with the rest of the data — if you'd already seen those extreme rows in the univariate pass, you'd catch it immediately instead of reporting a spurious relationship. This is also where you check for leakage: a feature that's suspiciously perfectly correlated with your target is often secretly derived from it.

## Step 4: Outliers, last and carefully

Outliers come last because you need the context from the earlier steps to judge them correctly. A $0 purchase amount is probably a data error. A $45,000 purchase in a dataset of $20-$200 transactions might be a legitimate wholesale order, a fraud case, or a typo with an extra zero — you can't tell which without looking at related columns (the customer's history, the product category) that the earlier steps already put in front of you. The practical move is to triage, not delete by default: tag each outlier as an error to fix, a rare-but-genuine case to keep, or a signal that one model won't serve your whole population well and you may need a segmented approach.

## Putting it together on a real pass

In practice this looks like: load the data, run `.shape`/`.dtypes`/`.isnull().sum()`, fix any obvious type problems, then loop through columns with `.describe()` and a histogram or `.value_counts()`, then build a correlation matrix and a few targeted scatterplots against your eventual target variable, then go back to the specific rows that stand out in the tails. Each pass is cheap on its own; skipping straight to modeling is what's expensive, because every bug you didn't catch in EDA resurfaces later as a confusing model result with no obvious cause.
