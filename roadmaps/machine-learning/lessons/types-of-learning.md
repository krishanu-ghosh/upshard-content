---
title: Types of learning
description: Supervised, unsupervised, semi-supervised, self-supervised and reinforcement learning — what distinguishes each, with a concrete example of every one.
roadmap: machine-learning
node: types-of-learning
group: foundations
video:
  provider: youtube
  id: "dGn0zAdJleY"
  start: 0
  end: null
creator:
  name: GeeksforGeeks
  channel_url: https://www.youtube.com/@GeeksforGeeksVideos
duration_seconds: 1593
level: beginner
prerequisites: [what-is-machine-learning]
takeaways:
  - Supervised learning learns from labelled examples, unsupervised learning finds structure with no labels at all, and reinforcement learning learns from the consequences of actions taken in an environment.
  - Semi-supervised learning uses a small amount of labelled data alongside a much larger pool of unlabelled data, which is the realistic situation for most real-world problems.
  - Self-supervised learning manufactures its own labels directly from the structure of the data — predicting a hidden word from its context, say — and is the technique behind how most large language models are pretrained.
check_questions:
  - q: What is the practical difference between supervised and semi-supervised learning?
    a: Supervised learning needs a label for every training example. Semi-supervised learning uses a small labelled set alongside a much larger unlabelled one, which matters because labelling data is usually the expensive, slow part of a real project.
  - q: How does self-supervised learning create labels without a human labelling anything?
    a: It constructs a prediction task directly from the data's own structure — hiding part of an input and asking the model to predict it, such as a missing word from the words around it. No person ever assigns a label; the data supplies its own.
deeper:
  - label: "Perfect Computer Engineer — Types of ML, including semi-supervised learning"
    url: https://www.youtube.com/watch?v=97HCQqvhK0Y
  - label: "Simplilearn — Supervised vs Unsupervised vs Reinforcement Learning"
    url: https://www.youtube.com/watch?v=1FZ0A1QCMWc
related_formulas: []
---

## Five categories, one underlying question

Every category of learning answers the same underlying question differently:
what kind of feedback does the model get while it learns? That single axis is
what separates all five categories covered here, and keeping it in view makes
the taxonomy far easier to hold onto than memorising five separate
definitions.

## Supervised learning: a label for every example

Supervised learning gets the most direct feedback possible: for every
training input, the correct output is given. Predicting a house's sale price
from its features, or classifying an email as spam or not, are both
supervised problems — the training data already contains the answer the
model is meant to produce. Nearly every lesson in the
[supervised models](/roadmaps/machine-learning/linear-regression) group of
this roadmap sits squarely in this category, and it is where most classical
machine learning work actually happens, because labelled data, where it
exists, makes the learning problem by far the most tractable of the five.

## Unsupervised learning: no labels at all

Unsupervised learning gets no answer key whatsoever — only raw inputs — and
has to find structure on its own: which points naturally group together, or
which few dimensions capture most of what varies across a dataset. There is
no "correct" clustering to check against, which is exactly why the
[unsupervised learning](/roadmaps/machine-learning/clustering) group later in
this roadmap spends real time on what it even means to evaluate a result
when no ground truth exists.

## Semi-supervised learning: the realistic middle

Most real projects do not land cleanly at either extreme. Labelling data
costs time and often needs a domain expert, so a project frequently ends up
with a small labelled set and a much larger pool of raw, unlabelled data
sitting unused. Semi-supervised learning uses both: the labelled portion
anchors the model to genuinely correct answers, while the much larger
unlabelled portion still contributes information about the overall shape of
the data, often substantially improving on what the small labelled set could
achieve alone.

## Self-supervised learning: inventing your own labels

Self-supervised learning solves the labelling problem by sidestepping it:
rather than a human assigning labels, the task is constructed directly from
the data's own structure. Hide a word in a sentence and ask a model to
predict it from the surrounding words — the sentence itself supplies the
correct answer, with no human labelling involved at all. This is, concretely,
how the language models behind Upshard's separate AI Engineer roadmap are
pretrained at the scale they operate at: there is nowhere near enough
human-labelled text in existence to train them the supervised way, so the
vast bulk of their pretraining is this self-supervised prediction task,
repeated across an enormous volume of ordinary text.

## Reinforcement learning: feedback from consequences

Reinforcement learning is the odd one out structurally: there is no dataset
of correct answers prepared in advance at all. Instead, an agent takes
actions inside an environment and receives a reward signal afterward,
learning over many attempts which actions tend to lead to good outcomes. A
program learning to play a game by trial and error, improving as it
accumulates experience of what wins and what loses, is the clearest everyday
example — covered in more depth in this roadmap's own
[reinforcement learning](/roadmaps/machine-learning/reinforcement-learning-basics)
lesson.
