---
title: Pandas and NumPy Workflows
description: "Real dataframe workflows: loading messy data, cleaning it, merging tables, and vectorizing math with numpy instead of looping row by row."
roadmap: ai-data-scientist
node: pandas-and-numpy-workflows
group: data-science-coding
video:
  provider: youtube
  id: "2uvysYbKdjM"
  start: 0
  end: null
creator:
  name: Keith Galli
  channel_url: https://www.youtube.com/@KeithGalli
duration_seconds: 5651
level: intermediate
prerequisites: [python-for-data-science, sql-for-data-analysis]
takeaways:
  - "`df.isna().sum()` is the first command to run on any new dataframe, because missing values propagate silently through later calculations until a result looks wrong for no obvious reason."
  - "`groupby` plus an aggregate (`df.groupby('col').mean()`) is the pandas equivalent of SQL's GROUP BY, and the two are worth translating between mentally."
  - "`merge` behaves like a SQL join and takes the same `how='inner'/'left'/'outer'` choice, with the same risk of row duplication when the join key isn't unique."
  - NumPy's vectorized operations replace row-by-row loops with array-wide math, which is both shorter to write and dramatically faster on real-sized data.
check_questions:
  - q: "Why run df.isna().sum() before doing anything else with a new dataframe?"
    a: "It shows exactly how many missing values are in each column up front, so you can decide deliberately how to handle them instead of discovering later that a mean or a merge quietly produced wrong numbers because of unnoticed NaNs."
  - q: "What's the difference between df.groupby('city')['sales'].sum() and df.groupby('city').sum()?"
    a: "The first sums only the sales column within each city group and returns a single series, while the second sums every numeric column in the dataframe within each group, which usually includes columns you didn't mean to aggregate."
  - q: "Why would merging two dataframes produce more rows than either dataframe had on its own?"
    a: "If the join key isn't unique in one of the dataframes, each matching row on one side gets paired with every matching row on the other side, multiplying rows the same way a non-unique join key does in SQL."
deeper:
  - label: Joining and Merging Dataframes
    url: "https://www.youtube.com/watch?v=XMjSGGej9y8"
related_formulas: []
---

## Loading and immediately interrogating your data

A dataframe loaded with `pd.read_csv("data.csv")` should never be trusted blindly. The first three commands to run, every time, are `df.shape` (how many rows and columns you actually got), `df.dtypes` (whether pandas guessed the right type for each column — a numeric column full of commas or currency symbols often gets loaded as text), and `df.isna().sum()` (how many missing values sit in each column). Skipping this step is the single most common cause of "my analysis gives a weird number" bugs later, because missing values and wrong types don't throw errors in pandas — they propagate quietly. A `NaN` in a column you later average just gets excluded from the mean by default, which can be exactly right or exactly wrong depending on why it's missing, and you won't know which unless you looked first.

`df.head()` and `df.sample(5)` are both worth running, not just one — `head()` shows you the start of the file, which is often cleaner than the middle, while `sample()` gives you a more honest look at what the bulk of the data actually contains.

## Cleaning: types, missing data, and the trap of chained assignment

Cleaning is mostly about getting types and missingness under control before doing math. `df['price'] = df['price'].str.replace('$', '').astype(float)` strips a currency symbol and converts a text column to numeric — a pattern you'll write constantly on real-world data. For missing values, you have three honest choices: drop the rows (`df.dropna(subset=['price'])`), fill them with a defensible value (`df['price'].fillna(df['price'].median())`), or leave them and make sure every downstream calculation accounts for them deliberately. The wrong choice is rarely "which of these three" — it's not making a conscious choice at all and letting pandas' defaults decide for you.

One gotcha worth knowing by name: chained assignment. Writing `df[df['price'] > 0]['price'] = df[df['price'] > 0]['price'].round(2)` looks reasonable but may silently fail to modify `df`, because the first bracket creates a copy pandas isn't guaranteed to let you write back into. The reliable pattern is `.loc`: `df.loc[df['price'] > 0, 'price'] = df.loc[df['price'] > 0, 'price'].round(2)`. Whenever you're filtering rows and then assigning into the result in the same breath, reach for `.loc` rather than chained bracket indexing.

## Grouping, merging, and translating from SQL

If you've written `GROUP BY` in SQL, pandas' `groupby` is the same idea with different syntax. `df.groupby('region')['revenue'].sum()` groups rows by region and sums revenue within each group — the SQL equivalent of `SELECT region, SUM(revenue) FROM df GROUP BY region`. A subtlety: `df.groupby('region').sum()` without selecting a column first sums every numeric column in the dataframe per group, which usually includes columns you didn't want summed (an ID column, a year column). Always select the specific column or columns you mean to aggregate before calling the aggregate function, the same discipline SQL forces on you by requiring an explicit `SELECT` list.

`merge` is pandas' join. `pd.merge(orders, customers, on='customer_id', how='left')` keeps every row from `orders` and attaches matching customer columns, filling `NaN` where there's no match — exactly a SQL `LEFT JOIN`. The same risk applies too: if `customer_id` isn't unique in `customers`, rows in `orders` get duplicated once per match. Check `customers['customer_id'].is_unique` before merging on it, and check `len(result)` against `len(orders)` after, the same sanity check you'd run on a SQL join.

## NumPy: thinking in arrays instead of loops

Underneath every pandas column is a NumPy array, and the habit that separates fast, readable data code from slow, clunky code is replacing loops with vectorized operations. If you want to convert a column of Celsius temperatures to Fahrenheit, writing `for i in range(len(df)): df.loc[i, 'f'] = df.loc[i, 'c'] * 9/5 + 32` works but is slow and verbose. The vectorized version, `df['f'] = df['c'] * 9/5 + 32`, applies the formula to the entire column at once — NumPy performs the multiplication and addition across the whole array in compiled code rather than looping in Python, which on a dataframe with a million rows is the difference between milliseconds and minutes.

This extends to conditional logic: `np.where(df['score'] >= 60, 'pass', 'fail')` creates a new array by checking a condition across every row at once, replacing a loop with an if-statement inside it. `np.select` handles multiple conditions in the same spirit when `np.where` alone would need awkward nesting. The general rule worth internalizing: if you catch yourself writing a Python `for` loop that touches a dataframe row by row to compute something simple, there's almost always a vectorized pandas or NumPy expression that does the same thing in one line and runs an order of magnitude faster.
