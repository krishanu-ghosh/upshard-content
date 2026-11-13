---
title: What is MLOps?
description: Why a model that works in a notebook still needs an entire operational discipline before it can be trusted in production.
roadmap: mlops
node: what-is-mlops
group: foundations
video:
  provider: youtube
  id: "biqYkVf-a7Y"
  start: 0
  end: null
creator:
  name: TechWorld with Nana
  channel_url: https://www.youtube.com/@TechWorldwithNana
duration_seconds: 1583
level: beginner
prerequisites: []
takeaways:
  - A model that scores well in a notebook is a prototype, not a product — getting it into production reliably is a separate, much larger engineering problem.
  - MLOps is the discipline of operationalizing machine learning — automating and monitoring the path from data to a trustworthy, running prediction service.
  - Unlike traditional software, an ML system can fail silently — it keeps answering requests, just with answers that quietly get worse, which is why operating one requires different practices than operating a normal app.
check_questions:
  - q: Why isn't a model that performs well in a notebook automatically ready for production?
    a: Because the notebook only proves the model can produce good predictions on a fixed dataset once, while production requires repeatable training, reliable serving, versioning, and ongoing monitoring that the notebook never had to handle.
  - q: What does MLOps add on top of the "DevOps" idea of automating build, test, and deploy?
    a: It extends automation and monitoring to the data and the model itself, not just the code, because a model's behavior depends on training data and learned parameters that regular DevOps pipelines have no concept of.
  - q: What is a key way an ML system can fail that a typical web service usually doesn't?
    a: It can keep returning successful, error-free responses while the predictions themselves silently degrade in quality, with no exception or status code to flag it.
deeper: []
related_formulas: []
---

## From notebook to production is a bigger gap than it looks

Training a model that performs well is often the easy part. A data
scientist can load a clean CSV, split it into train and test sets, fit a
model, and watch accuracy climb past 90% — all in an afternoon, inside a
single notebook. The temptation is to treat this as "done." It isn't. That
notebook run proves the model *can* learn the pattern in that dataset, on
that day, with that exact preprocessing code run top to bottom in order. It
proves nothing about whether the model can be retrained next month when the
data has shifted, whether it can serve predictions to real users within an
acceptable latency, whether the exact preprocessing steps can be
reproduced by someone other than the person who wrote them, or whether
anyone will notice if its accuracy quietly drops six weeks from now.

This gap — between "a model that performed well once" and "a model you can
depend on" — is what MLOps exists to close. MLOps (Machine Learning
Operations) is the set of practices, tooling, and automation that takes a
model from a one-off experiment to a system that trains, deploys, serves,
and retrains itself reliably, with humans supervising rather than
hand-running every step.

## The DevOps analogy, and where it breaks down

MLOps borrows its name and much of its mindset from DevOps, the discipline
that transformed how software gets shipped. Before DevOps, deploying
software often meant a manual, error-prone handoff from developers to a
separate operations team, with long release cycles and frequent "it worked
on my machine" failures. DevOps fixed this with automated testing,
continuous integration and delivery (CI/CD), and treating infrastructure as
version-controlled code — so changes could be shipped often, safely, and
reproducibly.

MLOps applies that same instinct — automate the repeatable parts, make
everything reproducible, catch problems before users do — to ML systems.
But it has to extend DevOps in a specific direction: in a normal software
system, the artifact you build, test, and deploy is code. In an ML system,
the artifact is a *model*, and a model is a function of both code and
data. Two engineers running the identical training script against
different snapshots of the same table can get two meaningfully different
models. That means versioning, testing, and reviewing can't stop at code —
they have to extend to datasets and to the trained model artifacts
themselves, each tied to the exact code and data that produced it. A
standard CI/CD pipeline has no built-in concept of "this model's accuracy
dropped 4 points against last week's holdout set" — that has to be built
deliberately into an ML pipeline.

## Why silent failure is the defining problem

The sharpest practical difference between operating ML systems and
operating ordinary software is how they fail. A web server that's
misconfigured typically throws errors, returns 500s, or crashes — loud,
visible, alerting-friendly failures. A model serving predictions can be
"up" in every conventional sense — 200 status codes, low latency, no
exceptions in the logs — while quietly giving worse and worse answers,
because the real-world data it now sees has drifted away from the data it
was trained on. A fraud model trained on last year's spending patterns
doesn't throw an error when this year's patterns shift; it just starts
missing more fraud, and nothing in a standard uptime dashboard will tell
you. Catching that requires monitoring the *statistical behavior* of
predictions and the *distribution* of incoming data, not just whether the
service responds — a kind of observability traditional ops tooling was
never built to provide.

## Why this sets up everything that follows

Every lesson after this one in this roadmap is really an answer to the
same question this lesson raises: once you accept that a trained model is
not a finished product, what has to be built around it to make it one?
You'll need a repeatable *lifecycle* connecting data to a deployed,
monitored model rather than a single notebook run. You'll need to be
precise about what makes operating ML genuinely different from operating
conventional software, beyond just "there's a model involved." And you'll
need to understand why a model that trained perfectly well can still
behave differently the moment it's serving real traffic. MLOps isn't a
single tool or a job title — it's the accumulated answer to "how do we
stop trusting a model just because it worked once."
