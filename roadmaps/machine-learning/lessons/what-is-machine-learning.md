---
title: What is machine learning?
description: A system that improves at a task from data rather than being explicitly programmed with rules — and how that differs from using a pretrained model.
roadmap: machine-learning
node: what-is-machine-learning
group: foundations
video:
  provider: youtube
  id: "9gGnTQTYNaE"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 503
level: beginner
prerequisites: []
takeaways:
  - Machine learning is a system that improves at a task by learning patterns from data, rather than following rules a programmer wrote by hand.
  - Traditional software encodes a human's understanding of a problem directly as logic; a trained model encodes whatever pattern the data happened to contain, which can be more powerful and less predictable at once.
  - Training a model is a different job from using one — this roadmap is about the former, in contrast to Upshard's "AI Engineer" roadmap, which is about building products on models someone else already trained.
check_questions:
  - q: What makes a system "machine learning" rather than ordinary software?
    a: The rules it follows were learned from examples rather than written by a programmer. The same training process, pointed at different data, produces different behaviour without anyone touching the code.
  - q: How does training a model differ from the work covered in Upshard's AI Engineer roadmap?
    a: Training adjusts a model's parameters from your own data. Using a pretrained model — what the AI Engineer roadmap covers — treats the model as a finished dependency and builds the context, tools and checks around it instead.
deeper:
  - label: "StatQuest — A Gentle Introduction to Machine Learning"
    url: https://www.youtube.com/watch?v=Gv9_4yMHFhI
  - label: "CrashCourse — Machine Learning & AI, Crash Course Computer Science #34"
    url: https://www.youtube.com/watch?v=z-EtmaFJieY
related_formulas: []
---

## Rules you write versus rules you discover

Ordinary software is a set of instructions a person wrote down in advance: if
the balance is below zero, decline the transaction; if the input matches this
pattern, return that result. The programmer already understood the rule, and
the code simply encodes it. Machine learning inverts this. Instead of writing
the rule, you show a model many examples of inputs paired with the right
outputs, and an algorithm searches for a rule that fits those examples well
enough to generalise to new ones it has never seen.

This is the right frame for everything that follows in this roadmap. A spam
filter does not consist of a list of banned words someone typed in; it
consists of parameters fit to millions of real emails, discovered by an
optimisation process rather than authored by a person. The upside is that it
can catch patterns a person would never think to write down. The downside,
which the rest of this roadmap spends considerable time on, is that nobody
wrote the rule, so nobody can read it off directly — which is exactly why
[evaluating a model](/roadmaps/machine-learning/evaluation-metrics) honestly
matters so much more here than in software whose logic a person can simply
read.

## Where this differs from being an "AI engineer"

It is worth being explicit about a distinction this roadmap's sibling,
Upshard's AI Engineer roadmap, is built entirely around. An AI engineer
treats a large language model as a finished, pretrained dependency — someone
else already ran the training, spent the compute, and shipped an API. The
engineering problem there is what you build around that dependency: context,
retrieval, tools, evaluation. This roadmap is about the other half of the
picture: how a model gets trained from data in the first place, using
classical methods that are often simpler, cheaper, and more interpretable
than a large language model, and are frequently the right tool when the
problem is narrower than "understand and generate human language."

Neither skill replaces the other. A team shipping an LLM-powered product
still needs someone who understands what a model actually is and how
training works, if only to know when a much simpler trained model would do
the job better, faster, and for a fraction of the cost. A fraud-detection
system flagging a transaction in milliseconds, for instance, is far more
likely to be a small trained classifier than a large language model — the
problem is narrow, the latency budget is tight, and a model built from this
roadmap's methods is both cheaper to run and easier to audit.

## The shape of what is ahead

This roadmap follows one coherent arc: preparing data a model can actually
learn from, the classical models that learn from labelled examples,
combining those models into ensembles, finding structure in data with no
labels at all, honestly evaluating whether any of it worked, and finally
where neural networks pick up from the classical foundation. Each group
answers one question in that arc, and each assumes nothing from the ones that
come after it — only from what came before.
