---
title: Imbalanced data
description: When one class vastly outnumbers another, why accuracy becomes a misleading metric, and the techniques — resampling, class weights — that address it.
roadmap: machine-learning
node: imbalanced-data
group: working-with-data
video:
  provider: youtube
  id: "GR-OW5asKlk"
  start: 0
  end: null
creator:
  name: Emma Ding
  channel_url: https://www.youtube.com/@emma_ding
duration_seconds: 823
level: intermediate
prerequisites: [feature-engineering]
takeaways:
  - When one class vastly outnumbers another, a model that always predicts the majority class achieves high accuracy while being completely useless, which is exactly why accuracy alone is a misleading metric here.
  - Resampling techniques change the training data itself — oversampling the minority class, undersampling the majority, or synthesising new minority examples with SMOTE — to give the model a more balanced view during training.
  - Class weights change the training objective instead of the data, penalising a mistake on the minority class more heavily than an identical mistake on the majority class.
check_questions:
  - q: Why can a model with 99% accuracy still be useless on an imbalanced dataset?
    a: If 99% of examples belong to one class, a model that always predicts that class achieves 99% accuracy while never correctly identifying a single example of the minority class — the number looks excellent and the model has learned nothing useful.
  - q: What is the difference between how resampling and class weights address imbalance?
    a: Resampling changes the training data itself, by duplicating, synthesising, or removing examples before training. Class weights leave the data untouched and instead change the cost function, making an error on the minority class more expensive to the model than the same error on the majority class.
deeper:
  - label: "Bhavesh Bhatt — SMOTE for Handling Imbalanced Datasets"
    url: https://www.youtube.com/watch?v=U3X98xZ4_no
  - label: "Bhavesh Bhatt — Class Weights for Handling Imbalanced Datasets"
    url: https://www.youtube.com/watch?v=Kp31wfHpG2c
related_formulas: []
---

## When accuracy stops meaning anything

Fraud detection, rare disease diagnosis, and manufacturing defect detection
share a structural feature: the thing being predicted is rare. If only one
in a thousand transactions is fraudulent, a model that predicts "not fraud"
for every single transaction achieves 99.9 percent accuracy while being
completely useless — it has never once correctly identified the case that
actually matters. This is not a hypothetical edge case; it is the default
outcome of training naively on imbalanced data without accounting for the
imbalance at all, because that trivial always-predict-the-majority model is
a genuine local optimum for plain accuracy.

This is exactly why [evaluation metrics](/roadmaps/machine-learning/evaluation-metrics) beyond raw accuracy — precision, recall, and the
tradeoff between them — matter enormously once a dataset is imbalanced, and
why this lesson sits right before that one.

## Resampling: changing the data

**Oversampling** duplicates examples of the minority class so it makes up a
larger share of the training set, at the risk of the model simply memorising
those duplicated examples rather than generalising from them.
**Undersampling** instead removes examples of the majority class, which
avoids duplication but throws away data that might otherwise have been
useful, and can hurt badly when the original dataset was not large to begin
with.

**SMOTE** (Synthetic Minority Oversampling Technique) improves on plain
duplication by generating new, synthetic minority examples rather than
copying existing ones — each synthetic point is created somewhere between two
real minority examples that are close to each other in feature space. This
gives the model new, plausible minority examples to learn from rather than
exact repeats, which generally helps it generalise better than duplication
alone.

## Class weights: changing the cost instead

Rather than touching the data at all, an alternative approach changes what
the model is being optimised to minimise. A standard training objective
treats every mistake equally; a weighted objective makes a mistake on the
minority class cost more than an identical mistake on the majority class,
directly countering the model's natural tendency to ignore a rare class in
favour of the much larger one. This has a practical advantage over resampling
— the original data is left completely untouched, with no duplicated or
synthetic examples introduced — though it requires a model and training
setup that actually supports weighted costs, which not every implementation
does equally well.

## The split itself needs the same care

Imbalance does not only distort training — it can distort evaluation too, if
the train/test split is done naively. A plain random split on a dataset
where the minority class makes up one percent of examples can, by chance,
leave the test set with almost none of that class at all, making any metric
computed on it essentially meaningless. **Stratified splitting** avoids this
by preserving the original class proportions in every split: if the full
dataset is one percent positive, both the training set and the test set are
built to be one percent positive too, rather than leaving that ratio to
chance. This matters most exactly when the imbalance is most severe, which is
precisely when a naive split is most likely to produce a test set that
badly misrepresents the minority class.

## Choosing the right metric is still the real fix

Both of these techniques improve what a model learns, but neither one matters
at all if the metric used to judge the model continues to be plain accuracy.
Precision, recall, and the balance between them, covered in the
[evaluation metrics](/roadmaps/machine-learning/evaluation-metrics) lesson
immediately following this one, are what actually reveal whether a model
handles the minority class well — and in a genuinely imbalanced problem, that
choice of metric usually matters more than which specific resampling
technique or weighting scheme gets used.
