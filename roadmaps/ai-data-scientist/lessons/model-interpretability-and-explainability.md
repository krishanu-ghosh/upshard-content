---
title: Model Interpretability and Explainability
description: How to explain feature importance, SHAP values, and partial dependence to a non-technical stakeholder without misrepresenting what a model actually does.
roadmap: ai-data-scientist
node: model-interpretability-and-explainability
group: applied-modeling
video:
  provider: youtube
  id: "MQ6fFDwjuco"
  start: 0
  end: null
creator:
  name: A Data Odyssey
  channel_url: https://www.youtube.com/@adataodyssey
duration_seconds: 426
level: intermediate
prerequisites: [overfitting-and-cross-validation]
takeaways:
  - Global feature importance tells you which features matter most across all predictions on average, but it says nothing about why any single prediction came out the way it did.
  - SHAP values explain individual predictions by fairly dividing credit for that specific output among the features that produced it, which is what a stakeholder asking "why did this customer get flagged" actually needs.
  - Partial dependence plots show how the model's prediction changes as one feature varies while others are held fixed, revealing the shape of a relationship rather than just its overall strength.
  - "Feature importance measures association the model learned, not causation in the real world; a feature can rank highly because it's a strong proxy for something else, not because changing it would change the outcome."
  - Translating these tools for a non-technical audience means leading with the concrete question they asked, not the method name, and showing direction and magnitude in plain terms.
check_questions:
  - q: What's the key difference between global feature importance and a SHAP explanation for one prediction?
    a: Global feature importance ranks features by their average effect across the whole dataset, while a SHAP explanation breaks down one specific prediction into how much each feature pushed it above or below the baseline for that one case.
  - q: What does a partial dependence plot show that a single feature importance number doesn't?
    a: It shows the shape of the relationship across the full range of a feature's values, such as risk rising sharply after a threshold and then leveling off, rather than just a single number summarizing overall importance.
  - q: Why is it a mistake to tell a stakeholder that a top-ranked feature "causes" the outcome?
    a: "Feature importance reflects a statistical association the model found in historical data, which can arise from a feature being a proxy for the real driver rather than a direct cause, so treating it as causal risks a wrong real-world decision."
deeper:
  - label: Feature Importance in Decision Trees
    url: "https://www.youtube.com/watch?v=6lDAlmGiWCM"
  - label: Interpretable vs Explainable Machine Learning
    url: "https://www.youtube.com/watch?v=VY7SCl_DFho"
related_formulas: []
---

## Two different questions stakeholders ask

When a model goes into production, stakeholders don't ask "is it accurate," they ask two more specific questions, and confusing them is the most common mistake in this part of the job. The first is "what does this model generally care about" — a product manager wondering why a churn model is worth trusting at all wants to know its overall drivers. The second is "why did this particular customer get this particular prediction" — a support agent facing an angry customer who was denied a discount needs an answer about that one case, not a dataset-wide summary. The first question is answered by global feature importance; the second is answered by per-prediction tools like SHAP. Picking the wrong tool for the question you were actually asked is how data scientists end up giving technically correct but unhelpful answers.

## Global feature importance: the view from 10,000 feet

Global feature importance ranks features by how much they contribute to the model's predictions on average across the whole dataset. For a tree-based model, a common version of this counts how much each feature reduces prediction error every time it's used for a split, summed across every tree and every split, then normalized so the scores are comparable. If "months since last login" comes out on top for a churn model, that tells you the model leans heavily on recency of activity to separate churners from non-churners, in aggregate. This is genuinely useful for sanity-checking a model before trusting it: if a model trained to predict loan default ranks "applicant's first name" as a top feature, that's a red flag that something is wrong in your data pipeline, not a reason to trust the model more. But global importance is an average, and averages hide variation. A feature can matter enormously for one segment of customers and not at all for another, and a single global number collapses that distinction away.

## SHAP values: explaining one prediction at a time

SHAP (SHapley Additive exPlanations) answers the per-case question by borrowing an idea from game theory: treat each feature as a "player" contributing to the "payout," which here is the gap between this specific prediction and the model's average prediction, then divide that gap fairly among the features based on how much each one moved the outcome. Concretely, if a churn model predicts an 80% churn probability for one customer against an average of 20% across all customers, a SHAP explanation might show that "40 days since last login" added 35 percentage points, "canceled a feature last month" added 20 points, and "long-tenured account" subtracted 5 points, netting out to the 80% prediction. This is exactly the shape of answer a support agent or a regulator asking "why was this specific decision made" needs, and it's additive and consistent in a way that makes it trustworthy to present: the contributions genuinely sum to the actual prediction, rather than being a loose approximation.

The practical cost is that SHAP values are computed per prediction and can be slow to generate for large models and datasets, so in practice teams compute them for individual cases on demand (a customer complaint, an audit request) rather than for every row of a million-row dataset, and use the faster global importance for routine model monitoring.

## Partial dependence: seeing the shape of a relationship

Both tools above summarize the overall effect of a feature with a number, but neither shows its shape. A partial dependence plot does: it holds every other feature fixed and sweeps one feature across its full range, plotting how the model's average prediction changes. For a loan default model, a partial dependence plot on "debt-to-income ratio" might reveal that predicted default risk is nearly flat below a 30% ratio, then rises sharply between 30% and 50%, then levels off above 50% because the model has already learned those applicants are high risk regardless of further increases. That threshold-and-plateau shape is a finding a single importance score can never show, and it's often exactly what a risk committee wants to see before approving a new lending policy, because it tells them where the model's sensitivity actually lives.

## Explaining this to someone who doesn't build models

When you present any of this to a non-technical stakeholder, lead with their actual question and the direction and size of the effect in plain language, and leave the method's name for a footnote, not the headline. Say "customers who haven't logged in for over a month are the single biggest driver of this prediction, accounting for most of why the model flagged them" rather than "the top SHAP feature was days_since_login with a value of 0.35." Use the partial dependence shape to answer "what would need to change" rather than abstract importance rankings, because that's the actionable version of the same information. And always flag, explicitly and without being asked, that these tools describe patterns the model learned from historical data, not proven causes in the world — a feature can rank high because it's correlated with the real driver rather than being the driver itself, and a stakeholder who walks away believing "if we just change this feature, the outcome changes" may make a costly decision based on a misunderstanding you could have prevented with one extra sentence.
