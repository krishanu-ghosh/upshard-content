---
title: Generalization and overfitting
description: Why a model that performs well on training data can fail on new data, the bias-variance tradeoff, and why train/validation/test splits exist.
roadmap: machine-learning
node: generalization-and-overfitting
group: foundations
video:
  provider: youtube
  id: "EuBBz3bI-aA"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 396
level: beginner
prerequisites: [the-ml-workflow]
takeaways:
  - A model that fits its training data extremely well has not necessarily learned anything useful — it may simply have memorised noise specific to that exact dataset.
  - Underfitting means the model is too simple to capture the real pattern; overfitting means it is complex enough to capture noise as if it were pattern. Both produce a model that performs poorly on new data, for opposite reasons.
  - A held-out test set exists for one reason only — to measure performance on data the model's training never touched, which is the only honest estimate of how it will do in the real world.
check_questions:
  - q: Why can a model with near-perfect accuracy on its training data still be a bad model?
    a: Because high training accuracy is consistent with having simply memorised that specific dataset, including its noise and quirks, rather than having learned the general pattern — which is exactly what overfitting looks like.
  - q: Why is it wrong to tune a model's settings using its final test set?
    a: Because every time you look at test performance and adjust something in response, you are fitting to that test set, which destroys its one purpose — giving an honest, untouched estimate of performance on data never used to influence the model in any way.
deeper:
  - label: "IBM Technology — Overfitting, Underfitting, and Bad Data"
    url: https://www.youtube.com/watch?v=0RT2Q0qwXSA
  - label: "deeplizard — Train, Test, & Validation Sets explained"
    url: https://www.youtube.com/watch?v=Zi-0rlM4RDs
related_formulas: []
---

## The question generalization actually asks

A model is only useful if it works on data it has never seen. That sounds
obvious, and it is routinely the thing projects get wrong: a model can fit
its training data almost perfectly and still be nearly useless, because what
it fit was partly or entirely noise specific to that one dataset rather than
a pattern that holds more generally. **Generalization** is the name for a
model's ability to carry its performance over to genuinely new data, and
essentially everything in this lesson is about why that transfer succeeds or
fails.

## Underfitting and overfitting, as opposite failures

**Underfitting** happens when a model is too simple to capture the real
pattern in the data at all — fitting a straight line to a relationship that
is clearly curved, for instance. It performs poorly on the training data
itself, which at least makes it easy to notice.

**Overfitting** is more dangerous precisely because it is harder to notice:
the model is complex enough to fit not just the real underlying pattern but
also the specific noise and coincidences present in this one training set. It
performs beautifully on training data and considerably worse on anything new,
because a good deal of what it "learned" was never a real pattern to begin
with — it was an artefact of exactly these examples.

## The bias-variance tradeoff

This failure pair has a formal name: the bias-variance tradeoff. **Bias** is
error from a model that is too simple — systematic, consistent mistakes
regardless of which particular training set it saw, which is the signature of
underfitting. **Variance** is error from a model that is too sensitive to the
specific training data it happened to receive — trained again on a slightly
different sample of the same underlying problem, it would produce
meaningfully different predictions, which is the signature of overfitting.

Model complexity moves these two in opposite directions: a simpler model
tends toward higher bias and lower variance, a more complex model tends
toward lower bias and higher variance, and the right amount of complexity for
a given problem sits somewhere between the two extremes rather than at
either end.

## Why the split exists

None of this would be detectable without deliberately holding some data back.
The standard practice splits data into three parts. The **training set** is
what the model actually learns from. The **validation set** is used while
developing the model, for comparing configurations and tuning settings, and
is revisited repeatedly during that process. The **test set** is touched
exactly once, at the very end, to report a final, honest estimate of how the
model will perform on data it has never influenced in any way.

The discipline that makes this work is treating the test set as genuinely
off-limits until that one final check. Looking at test performance and then
adjusting anything in response — a parameter, a feature, a choice between
models — quietly turns the test set into a second validation set. The honest
estimate it was supposed to provide is gone. This is also precisely
why a single split, however carefully protected, can still give a
misleading read if that one particular split happened to be unusually easy
or unusually hard — which is the problem
[cross-validation](/roadmaps/machine-learning/cross-validation) exists to
solve later in this roadmap.
