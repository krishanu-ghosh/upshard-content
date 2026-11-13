---
title: Evaluation metrics
description: Accuracy, precision, recall and the confusion matrix for classification; MSE, RMSE and R² for regression — and why the right metric depends on the problem.
roadmap: machine-learning
node: evaluation-metrics
group: evaluating-models
video:
  provider: youtube
  id: "Kdsp6soqA7o"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 432
level: intermediate
prerequisites: [logistic-regression]
takeaways:
  - The confusion matrix breaks every prediction into one of four outcomes — true positive, true negative, false positive, false negative — and every classification metric is built from these four counts.
  - Precision asks what fraction of positive predictions were actually correct; recall asks what fraction of actual positives were correctly found. Improving one typically costs the other.
  - For regression, MSE and RMSE penalize large errors disproportionately; R² reports what fraction of the variance in the outcome the model actually explains, on a scale that is comparable across different problems.
check_questions:
  - q: Why does a model's accuracy alone fail to reveal that it performs badly on a rare but important class?
    a: Accuracy counts every correct prediction equally, so a model that is right nearly all the time on the common class and wrong almost all the time on the rare one can still post a very high overall accuracy, because the rare class barely counts toward the total.
  - q: When would you prioritize recall over precision, or the reverse?
    a: Prioritize recall when missing a true positive is costly — a missed cancer diagnosis, say — even if that means tolerating more false alarms. Prioritize precision when a false alarm is costly and missing some true positives is more tolerable, such as a system where every flagged case triggers an expensive manual review.
deeper:
  - label: "StatQuest — ROC and AUC, Clearly Explained"
    url: https://www.youtube.com/watch?v=4jRBRDbJemM
  - label: "StatQuest — R-squared, Clearly Explained"
    url: https://www.youtube.com/watch?v=2AQKmw14mHM
related_formulas: []
---

## Why a single accuracy number is not enough

As the [imbalanced data](/roadmaps/machine-learning/imbalanced-data) lesson
established, a model can post a very high accuracy while being nearly useless
on the class that actually matters. Evaluation metrics exist precisely to
look past that single aggregate number and ask more specific, more honest
questions about what a model actually gets right and wrong.

## The confusion matrix: where everything else starts

Every classification outcome falls into exactly one of four categories. A
**true positive** is a correct positive prediction; a **true negative** is a
correct negative prediction. A **false positive** is predicting positive when
the truth was negative — a false alarm. A **false negative** is predicting
negative when the truth was positive — a miss. The **confusion matrix** is
simply a table of how many predictions fell into each of these four
categories, and nearly every classification metric that follows is computed
directly from these four counts.

## Precision and recall: two different questions

**Precision** asks: of everything the model predicted as positive, what
fraction actually was? It is true positives divided by the sum of true
positives and false positives: a measure of how trustworthy a positive
prediction is. **Recall** asks a different question: of everything that was
actually positive, what fraction did the model correctly find? It is true
positives divided by the sum of true positives and false negatives: a
measure of how thorough the model is at catching real positives.

These two typically trade against each other. A model can achieve perfect
recall trivially by predicting positive for everything, at the cost of
terrible precision, since it would also flag every true negative as a false
positive. It can achieve high precision by predicting positive only when
extremely confident, at the cost of missing many true positives and so
lowering recall. The **F1 score** combines both into one number, their
harmonic mean. It's a useful single summary when both matter and neither
should be allowed to be very weak while the other is strong. But it hides
which of the two is doing the work — looking at precision and recall
separately usually tells you more than the combined score alone.

## Which one to prioritize depends on the cost of each mistake

There is no universally correct choice between prioritising precision or
recall — it depends entirely on which kind of mistake costs more in the
specific application. A missed medical diagnosis is usually far more costly
than an unnecessary follow-up test, arguing for prioritising recall even at
the expense of precision. An automated system that triggers an expensive
manual review on every flagged case needs its false-alarm rate controlled
carefully, arguing for prioritising precision instead. This decision connects
directly back to the decision-threshold choice covered in
[logistic regression](/roadmaps/machine-learning/logistic-regression) —
moving that threshold is exactly how precision and recall get traded against
each other in practice.

## Regression metrics: measuring continuous error

For predicting a continuous value rather than a category, **mean squared
error (MSE)** averages the squared difference between predictions and actual
values — the same quantity linear regression is trained to minimise, covered
in [linear regression](/roadmaps/machine-learning/linear-regression).
**RMSE**, its square root, converts that error back into the original units
of the outcome, which makes it much easier to interpret directly. An RMSE of
five dollars on a price-prediction problem is immediately meaningful in a way
a squared-dollars number is not. **MAE** (mean absolute error) averages the
plain absolute difference instead of the squared one, which penalises large
errors less severely than MSE or RMSE do.

**R²** answers a different question entirely: what fraction of the variance
in the outcome does the model actually explain, on a scale from zero
(explains none of it, no better than always predicting the average) to one
(explains all of it perfectly)? Because it is a standardised fraction rather
than a raw error in the outcome's own units, R² is comparable across entirely
different problems in a way that RMSE, tied to whatever units the original
outcome happens to be measured in, simply is not.
