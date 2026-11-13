---
title: The ML workflow
description: Collect, explore, engineer features, split, train, evaluate, tune, deploy, monitor — and why the real version of this loop is iterative, not a straight line.
roadmap: machine-learning
node: the-ml-workflow
group: foundations
video:
  provider: youtube
  id: "-x9bVcEmkUk"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 326
level: beginner
prerequisites: [types-of-learning]
takeaways:
  - The workflow has recognisable stages — collect, explore, prepare, train, evaluate, tune, deploy, monitor — but real projects move backward through them constantly rather than following them once in order.
  - A model's work does not end at deployment; data drifts, the world changes, and a model that was accurate at launch degrades without anyone touching its code.
  - Most of a real project's time is spent on the data stages before training ever starts, not on the model itself.
check_questions:
  - q: Why is the ML workflow drawn as a loop rather than a straight line?
    a: Because a problem discovered at any stage usually sends you back to an earlier one — a model performing badly in evaluation often means returning to feature engineering, not just trying a different model.
  - q: What happens to a model's accuracy after it is deployed, if nothing else changes?
    a: It can still degrade, because the real-world data the model sees in production drifts away from the data it was trained on over time — a problem monitoring exists specifically to catch.
deeper:
  - label: "IBM Technology — Five Steps to Create a New AI Model"
    url: https://www.youtube.com/watch?v=jcgaNrC4ElU
  - label: "Gate Smashers — New to ML? Follow These Steps to Build Any Machine Learning Model"
    url: https://www.youtube.com/watch?v=fmxmNwAB0_Y
related_formulas: []
---

## The stages, named plainly

**Collect** the data the problem actually needs. **Explore** it before doing
anything else, covered in its own
[lesson](/roadmaps/machine-learning/exploratory-data-analysis). **Prepare**
it — cleaning, engineering features, scaling, splitting into training and
test sets — which is the entire next group of this roadmap. **Train** a
model on the prepared training data. **Evaluate** it honestly against data it
has not seen. **Tune** whatever did not work well enough, and often return to
an earlier stage entirely. **Deploy** the model somewhere it actually serves
predictions. **Monitor** it, because deployment is not the finish line.

Naming the stages in order makes the workflow sound like a straight line.
Real projects almost never move through it that way.

## Why it loops instead of running straight through

A model that performs poorly during evaluation rarely means "try a different
model" as the first response. Far more often it means returning to feature
engineering, because the signal the model needs simply is not present in the
current features — no amount of a more sophisticated model will recover
information that was never captured from the raw data in the first place.
Discovering a labelling error during evaluation sends you back to data
cleaning. A surprising result during exploratory analysis sends you back to
collection, because it reveals the data gathered does not actually represent
the problem at hand. The workflow is better understood as a graph with
backward edges than as a pipeline with one direction, and treating it as a
strict sequence is a reliable way to waste time chasing model improvements
that a data fix would have solved in a fraction of the effort.

## Why deployment is not the finish line

A model is fit to a snapshot of the world at training time. The real world
keeps moving: user behaviour shifts, a product changes, a market responds to
something the model never saw during training. This is usually called **data
drift**, and its consequence is blunt — a model's accuracy can fall steadily
after launch with nobody having touched a single line of its code, simply
because the data it now sees no longer resembles the data it learned from.

Monitoring a deployed model for this kind of drift, and having a plan for
retraining it on fresh data when drift is detected, is as much a part of the
real workflow as the training step itself — which is exactly why "monitor"
is a stage in the loop and not an afterthought tacked onto the end of it.

## Where most of the time actually goes

A common surprise for anyone new to this field: the model-training step
itself, which gets the most attention in courses and tutorials, is frequently
the smallest fraction of a real project's total time. Collecting, exploring
and preparing data reliably consumes the majority of the effort, which is
exactly why this roadmap devotes an entire group,
[preparing data a model can learn from](/roadmaps/machine-learning/exploratory-data-analysis),
to that work before a single model gets trained on any of it.
