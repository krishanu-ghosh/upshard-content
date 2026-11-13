---
title: Overfitting and Cross-Validation
description: How to spot overfitting in practice and use train/validation/test splits and cross-validation to get an honest read on model performance.
roadmap: ai-data-scientist
node: overfitting-and-cross-validation
group: applied-modeling
video:
  provider: youtube
  id: "fSytzGwwBVw"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 364
level: intermediate
prerequisites: [model-evaluation-and-metrics]
takeaways:
  - "Overfitting means a model has memorized noise and quirks specific to the training data, so it scores well there but performs noticeably worse on new, unseen data."
  - The clearest practical sign of overfitting is a large gap between training performance and validation performance, not training performance alone.
  - A three-way split into train, validation, and test sets exists because tuning decisions made using validation performance leak information, so a final, untouched test set is needed for an honest final number.
  - K-fold cross-validation reuses the data more efficiently than a single split and gives a more stable estimate of performance, which matters most when labeled data is limited.
  - Regularization, simpler models, more data, and early stopping are the standard levers for reducing overfitting once you've confirmed it's happening.
check_questions:
  - q: What is the clearest practical sign that a model is overfitting?
    a: "A large gap between how well it performs on the training data versus on held-out validation data: high training score with much lower validation score is the signature of memorized noise rather than learned patterns."
  - q: Why do you need both a validation set and a separate test set, rather than just one held-out set?
    a: Because you repeatedly use validation performance to make tuning decisions, information about the validation set leaks into the model indirectly through those choices, so a final test set that was never used for any decision is needed to get an honest, unbiased estimate of real-world performance.
  - q: What problem does k-fold cross-validation solve that a single train/validation split doesn't?
    a: A single split gives a performance estimate that depends on which particular rows happened to land in the validation set, which is unstable with limited data; k-fold cross-validation rotates every row through the validation role once and averages the results, giving a more reliable estimate.
deeper:
  - label: Machine Learning Fundamentals, Bias and Variance
    url: "https://www.youtube.com/watch?v=EuBBz3bI-aA"
related_formulas: []
---

## What overfitting looks like when you actually see it

Overfitting is when a model learns the specific noise and idiosyncrasies of its training data rather than the general pattern you actually want it to learn. It shows up concretely as a gap: the model scores 98% accuracy (or a tiny RMSE) on the data it was trained on, but drops to 75% accuracy (or a much larger RMSE) on data it hasn't seen before. The training score alone tells you almost nothing about whether this has happened — a sufficiently flexible model can always fit its training data near-perfectly, including the random noise in it, which is exactly the problem. You only detect overfitting by comparing performance on data the model was trained on against performance on data it wasn't, which is the entire reason held-out evaluation exists.

A concrete example makes this tangible. Say you're predicting house prices from 15 features using 200 training rows. A deep decision tree can split the data again and again until every single training house sits in its own leaf, reproducing the training prices exactly — training error near zero. But those final splits aren't capturing "houses near good schools cost more"; they're capturing "this specific house, numbered 147, with this specific combination of twelve micro-features, sold for exactly $412,000," a fact that has zero predictive value for the next house that walks through the door. On a fresh set of houses, that same tree performs far worse than a shallower one that stopped splitting earlier, because the shallow tree captured the real pattern and left the noise alone.

## Why a validation set alone isn't enough

The standard fix is to hold out data the model never trains on, and check performance there. But a single train/validation split has a subtle leak: you use validation performance to make decisions — which model family to use, how deep to let a tree grow, which features to keep — and every one of those decisions is implicitly tuned to do well on that specific validation set. After enough rounds of tuning, your validation score stops being an honest estimate of performance on truly new data, because you've effectively overfit to the validation set through your sequence of choices, even without the model ever training on it directly.

The fix for the fix is a three-way split: train, validation, and test. You train on the training set, tune every decision using the validation set, and once you're completely done tuning, you run the final model on the test set exactly once and report that number. The test set is untouched by every decision made up to that point, so its score is the most honest estimate you'll get of how the model performs on new data. The discipline that matters here is behavioral, not technical: the test set has to stay locked away until the very end, because peeking at it even once to "just check" and then going back to tune further defeats its entire purpose.

## Cross-validation: using limited data more efficiently

A single train/validation split has a second weakness beyond leakage: with limited data, which specific rows end up in the validation set is partly luck, and a different random split could give you a noticeably different validation score. K-fold cross-validation addresses this by splitting the training data into k roughly equal chunks (5 and 10 are common choices), training k separate times, each time using a different chunk as validation and the rest as training, and then averaging the k validation scores. Every row gets used for validation exactly once and for training k-1 times, so you're squeezing more signal out of the same data while also getting a performance estimate that isn't dependent on one lucky or unlucky split. The trade-off is compute: you're training the model k times instead of once, which matters when a single training run is already slow. In practice, cross-validation is used during the tuning phase, and a separate, never-touched test set is still held out for the final honest number, so the three-way split and cross-validation work together rather than replacing each other.

## What to actually do once you've confirmed overfitting

Once the train-validation gap confirms overfitting, there are a handful of standard levers, and picking among them is itself a judgment call. Regularization (L1 or L2 penalties in linear models, or pruning and minimum-leaf-size constraints in trees) directly discourages the model from fitting noise by penalizing complexity. Reducing model complexity — fewer features, a shallower tree, fewer boosting rounds — has the same effect through a different mechanism. Getting more training data helps because noise doesn't repeat consistently across more examples while real patterns do, so a larger dataset makes memorizing noise less rewarding relative to learning the actual signal. For models trained iteratively, like gradient boosting or neural networks, early stopping halts training once validation performance stops improving, even though training performance would keep climbing if you let it run longer. None of these levers is universally correct; the right one depends on whether the problem is "too complex a model for too little data" or "genuinely needs more complexity but the data is noisy," and the gap size, the learning curve shape, and domain knowledge together tell you which situation you're actually in.
