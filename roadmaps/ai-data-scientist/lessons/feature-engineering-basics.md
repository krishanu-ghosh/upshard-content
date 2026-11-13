---
title: Feature Engineering Basics
description: Turning raw columns into model-ready features through encoding, scaling, binning, and interaction terms, without leaking the target.
roadmap: ai-data-scientist
node: feature-engineering-basics
group: exploratory-data-analysis
video:
  provider: youtube
  id: "GduT2ZCc26E"
  start: 0
  end: null
creator:
  name: Greg Hogg
  channel_url: https://www.youtube.com/@GregHogg
duration_seconds: 2877
level: intermediate
prerequisites: [data-cleaning-and-missing-data]
takeaways:
  - Encoding choice depends on whether a categorical variable has a real order — label encoding is appropriate for ordinal categories, but using it on a nominal category invents a false numeric ranking the model will treat as real.
  - Scaling matters for distance-based and gradient-based models (k-NN, SVMs, neural networks, regularized regression) but is irrelevant to tree-based models, which split on thresholds rather than distances.
  - Binning and interaction terms let a simple model capture non-linear and combined effects it couldn't otherwise represent, at the cost of extra features and a real risk of overfitting if used without care.
  - Any feature built from statistics of the data — a scaler's mean and standard deviation, or a target-encoded category average — must be fit only on training data, or it leaks test-set information into the model and inflates your evaluation.
check_questions:
  - q: When is label encoding appropriate, and when does it cause a problem?
    a: Label encoding is appropriate for ordinal categories that have a genuine order, like a satisfaction rating from low to high, but it causes a problem on nominal categories like city names, where the arbitrary numbers assigned imply a false ranking that many models will try to learn from.
  - q: Why don't tree-based models need feature scaling?
    a: Because a decision tree splits a feature at a threshold value rather than computing a distance or a gradient across features, so whether a feature ranges from 0 to 1 or 0 to 1,000,000 has no effect on where the best split falls.
  - q: Why must a scaler or target encoder be fit only on the training set?
    a: "Because fitting it on the full dataset lets information from the test set, like its mean, standard deviation, or target averages, leak into the features the model trains on, which makes evaluation metrics look better than the model will actually perform on genuinely unseen data."
deeper:
  - label: "One-Hot, Label, Target and K-Fold Target Encoding, Clearly Explained!!!"
    url: "https://www.youtube.com/watch?v=589nCGeWG1w"
related_formulas: []
---

## Why raw columns usually aren't model-ready

Most machine learning models expect numeric input on a consistent scale. Most raw datasets don't arrive that way. A column might be text categories, a wildly skewed dollar amount, or a date that tells a model nothing by itself until it's broken into day-of-week or days-since-signup. Feature engineering is the work of reshaping raw columns into a form that exposes the signal a model can actually use. It's also where a lot of real modeling skill lives. Two people with the same algorithm and the same raw data can get meaningfully different results purely from how they engineered features.

## Encoding categorical variables correctly

Categorical variables need to become numbers, and the right method depends on whether the category has order. **Ordinal** categories have a real ranking — a survey response from "strongly disagree" to "strongly agree," or a shirt size from S to XL. Label encoding (mapping each category to an integer that preserves that order) is the right tool here: a model is correct to treat "agree" as between "neutral" and "strongly agree."

**Nominal** categories have no order: city, color, payment method. Label-encoding these invents a false relationship. City A becomes 0, city B becomes 1, city C becomes 2, and a model will quietly learn to treat city C as somehow "more" than city A. One-hot encoding is the standard fix for nominal categories: create one binary column per category, so each city gets its own 0/1 flag with no implied ranking.

The tradeoff is dimensionality. A `zip_code` column with 30,000 distinct values turns into 30,000 columns, which is often impractical. For high-cardinality categories like that, target encoding is a common alternative: replace each category with the average target value for that category, such as the average purchase amount for that zip code. Target encoding is powerful but leak-prone. It's built from the outcome you're trying to predict, so it must be computed only from training folds — typically with k-fold or leave-one-out encoding — or the model effectively sees the answer before it's asked the question.

## Scaling, and why it's conditional

Scaling transforms a numeric feature onto a common range, usually through standardization (subtract the mean, divide by the standard deviation, giving a feature mean 0 and standard deviation 1) or min-max normalization (rescale to a fixed range like 0 to 1). Whether this matters depends entirely on the model.

Distance-based models like k-nearest neighbors or SVMs compute distances across all features at once. A feature measured in the tens of thousands (annual income) will dominate a feature measured in single digits (number of children) purely because of its scale, not because it's actually more predictive. Scaling puts both on equal footing. Gradient-based models, including neural networks and regularized linear regression, benefit for a related reason: unscaled features with very different ranges distort the optimization surface and make regularization penalties unfair across features of different magnitude.

Tree-based models are the exception — decision trees, random forests, gradient boosting. A tree picks a threshold to split a feature on, and that threshold-finding process doesn't care whether the feature ranges from 0 to 1 or 0 to 1,000,000. Scaling is wasted effort there, though it's harmless if you do it anyway.

## Binning and interaction terms

Binning converts a continuous variable into discrete buckets, turning exact age into bands like 18-25, 26-35, 36-50, 50+. This helps when the real-world relationship between a feature and the target isn't linear or even monotonic. Customer churn might be high for very new accounts, low in the middle of a relationship, and creep up again for very old accounts nearing contract renewal — a pattern a plain linear model can't represent in the raw "months as customer" number, but can represent once that number is binned into categories. The cost is information loss: two customers at 25 and 34 months get treated identically if they fall in the same bin. Binning is a deliberate tradeoff, not a free improvement.

Interaction terms capture the case where two features only matter *together*. Think of a house listing: `square_footage` and `bedroom_count` each predict price on their own, but their ratio captures something neither does alone. A small house with many bedrooms, meaning cramped rooms, is a different kind of property than a large house with the same bedroom count. Creating an explicit interaction feature — multiplying two columns, or computing a ratio — lets a model that can't discover interactions on its own, like plain linear regression, benefit from a relationship a human noticed by understanding the domain. Tree-based models can discover some interactions automatically through successive splits, but even they benefit from an explicit interaction term when the true relationship is a specific multiplicative or ratio structure that would otherwise take many splits to approximate.

## The leakage trap that undoes all of this

Every technique above shares one failure mode: fitting on data the model shouldn't see yet. Think of it like grading a student on a test they've already seen the answer key for — the score looks great and tells you nothing real. A `StandardScaler` fit on the entire dataset before a train/test split bakes the test set's mean and standard deviation into every scaled value, including the training rows. The model ends up training on features that already encode information about the data it will later be "tested" on. The same trap applies to target encoding, binning thresholds chosen by looking at the full dataset, and any imputation value computed globally.

The fix is always the same discipline: split your data first, fit every transformation only on the training partition, then apply that already-fitted transformation to the test partition unchanged. It's a small procedural detail, but skipping it is one of the most common reasons a model that looks great in evaluation performs noticeably worse once it's actually deployed.
