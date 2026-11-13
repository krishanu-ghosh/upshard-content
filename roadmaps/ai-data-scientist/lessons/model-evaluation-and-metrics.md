---
title: Model Evaluation and Metrics
description: Why accuracy is often the wrong default metric, and how to choose precision, recall, F1, AUC, or RMSE based on what the model's mistakes actually cost.
roadmap: ai-data-scientist
node: model-evaluation-and-metrics
group: applied-modeling
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
prerequisites: [supervised-learning-for-prediction]
takeaways:
  - "Accuracy is misleading on imbalanced data, because a model that always predicts the majority class can score high accuracy while catching none of the cases that matter."
  - Precision and recall measure different mistakes; precision penalizes false alarms, recall penalizes missed cases, and improving one usually costs the other.
  - F1 and the precision-recall curve are better defaults than accuracy whenever the positive class is rare or the two error types have very different costs.
  - AUC-ROC summarizes a classifier's ranking ability across every possible threshold, which is useful for comparing models but doesn't tell you what threshold to actually use in production.
  - For regression problems, RMSE punishes large errors disproportionately while MAE treats every unit of error equally, so the choice between them should match how the business actually feels an error.
check_questions:
  - q: Why can a model with 95% accuracy still be useless in practice?
    a: "If only 5% of cases are positive, a model that predicts \"negative\" for everything scores 95% accuracy while catching zero of the cases you actually care about, which is exactly what high accuracy can hide on imbalanced data."
  - q: What is the difference between precision and recall, and why does improving one often hurt the other?
    a: Precision asks what fraction of predicted positives were actually positive, while recall asks what fraction of actual positives were caught; raising the decision threshold tends to improve precision but lower recall because the model becomes choosier.
  - q: When would you choose RMSE over MAE for a regression problem?
    a: Choose RMSE when large errors are disproportionately costly, such as a demand forecast that triggers an expensive stockout, since RMSE squares errors before averaging and therefore weighs big misses much more heavily than small ones.
deeper:
  - label: Machine Learning Fundamentals, Sensitivity and Specificity
    url: "https://www.youtube.com/watch?v=vP06aMoz4v8"
  - label: ROC and AUC, Clearly Explained!
    url: "https://www.youtube.com/watch?v=4jRBRDbJemM"
related_formulas: []
---

## Why accuracy fails as a default

Accuracy, the fraction of predictions a model gets right, feels like the natural metric because it's the easiest to explain. It's also the metric most likely to mislead you, and the failure mode is specific: imbalanced classes. Suppose you're building a model to flag fraudulent transactions, and fraud makes up 0.5% of all transactions. A model that predicts "not fraud" for every single transaction scores 99.5% accuracy while catching zero fraud cases, which is the opposite of useful. This isn't a contrived edge case; churn, disease screening, equipment failure, and fraud detection are all naturally imbalanced problems, and they're exactly the problems data scientists are most often asked to solve. The practical rule is: the moment one class is meaningfully rarer than the other, stop treating accuracy as the headline number, and look at what the model does specifically with the rare, important class.

## Precision and recall measure different mistakes

To see past accuracy, break predictions into a confusion matrix: true positives, false positives, true negatives, false negatives. Precision answers "of the cases I flagged as positive, how many actually were?" — it's hurt by false positives. Recall answers "of the actual positives, how many did I catch?" — it's hurt by false negatives. These pull in opposite directions because of how classifiers work: most produce a probability score, and you pick a threshold above which you call something positive. Lower that threshold and you catch more true positives (recall goes up) but also sweep in more false alarms (precision goes down). Raise it and the reverse happens. There is no setting that maximizes both at once, which is why the right balance depends entirely on what each type of mistake costs.

Consider two deployments of the same fraud model. A bank that auto-blocks flagged transactions needs high precision, because a false positive means blocking a legitimate purchase and angering a customer — an expensive, visible error. A bank that merely routes flagged transactions to a human reviewer can tolerate lower precision in exchange for higher recall, because a missed fraud case (false negative) costs real money while a false positive just costs a reviewer a few minutes. Same model, same scores, different acceptable threshold — and that threshold decision is a business call the data scientist should frame, not something a default library setting should make silently.

## F1, precision-recall curves, and AUC-ROC

When you need one number instead of a pair, F1 is the harmonic mean of precision and recall, and it only scores high when both are reasonably high — it's a good default summary metric whenever the positive class is rare and you don't yet know your cost trade-off. For a fuller picture across every possible threshold, plot a precision-recall curve, which shows the trade-off directly and lets you or a stakeholder pick a threshold with eyes open rather than accepting whatever a library defaults to (usually 0.5, which has no special justification for your problem).

AUC-ROC is a different kind of summary: it measures how well the model ranks positives above negatives across every threshold, independent of class balance in a particular way that makes it popular for model comparison. An AUC of 0.5 means the model ranks no better than random; 1.0 means perfect ranking. The catch is that AUC-ROC can look deceptively strong on heavily imbalanced data, because the false positive rate it uses in its denominator is diluted by a huge number of true negatives. For severely imbalanced problems, a precision-recall curve is usually more honest than ROC, and AUC-ROC is best used to compare two candidate models' general ranking ability, not as the metric you report as "how good is this model in production."

## Choosing metrics for regression problems

Regression problems need a different vocabulary, but the same underlying question: what does an error actually cost? Mean Absolute Error (MAE) averages the absolute size of every error in the original units — if you're predicting delivery time in minutes, an MAE of 8 means your typical prediction is off by 8 minutes, and every minute of error counts the same whether it's the first or the fiftieth. Root Mean Squared Error (RMSE) squares errors before averaging and then takes the square root, which means a single 50-minute miss contributes far more to RMSE than five 10-minute misses would, even though the five together add up to the same total error. Use RMSE when large errors are disproportionately damaging — a demand forecast that's off by a little is a rounding error, but one that's off by a lot triggers a stockout or a wasted shipment. Use MAE when errors scale linearly with cost and you want a number that's easy to explain in plain units. Reporting both, alongside a quick look at the distribution of errors (not just their average), catches the case where a model looks fine on average but is wildly wrong on a specific, important subset of cases.
