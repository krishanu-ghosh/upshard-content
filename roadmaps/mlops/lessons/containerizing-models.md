---
title: Containerizing models
description: Packaging a trained model and its dependencies into a Docker image so it runs identically on your laptop, in CI, and in production.
roadmap: mlops
node: containerizing-models
group: packaging
video:
  provider: youtube
  id: "h5wLuVDr0oc"
  start: 0
  end: null
creator:
  name: AssemblyAI
  channel_url: https://www.youtube.com/@AssemblyAI
duration_seconds: 1125
level: intermediate
prerequisites: [model-registries]
takeaways:
  - A container image bundles a model's weights, its exact library versions, and a minimal OS layer into one artifact, so "it works on my machine" stops being a meaningful excuse.
  - Docker images are built in layers, and ordering instructions from least-changing to most-changing (OS and dependencies first, model code and weights last) is what makes rebuilds and CI fast instead of slow.
  - A registry-pulled model version and a container image are two different artifacts with two different lifecycles, and conflating them is a common source of silent production drift.
  - Treating the container as the unit of deployment, not the raw model file, is what lets a serving platform schedule, scale, and roll back a model the same way it does any other service.
check_questions:
  - q: Why is a trained model file alone not a sufficient deployment artifact?
    a: Because the file has no record of the exact runtime, library versions, and OS packages it needs, so the same weights can produce different behavior or outright crash on a different machine.
  - q: Why does instruction order matter inside a Dockerfile?
    a: Docker caches each layer and only rebuilds layers after the first one that changed, so putting rarely-changing steps like dependency installation before frequently-changing steps like copying model code keeps most rebuilds fast.
  - q: What is the practical difference between a model registry and a container image?
    a: A model registry versions the trained weights and their metadata, while a container image versions the full runtime environment those weights run inside, and a deployable build pins a specific version of each.
deeper:
  - label: "Build and Run a Docker Container for your Machine Learning Model — Data Science Garage"
    url: "https://www.youtube.com/watch?v=mUnrWn6flfc"
related_formulas: []
---

## The artifact problem a trained model doesn't solve

A trained model, by itself, is just a file of numbers — weights, and maybe
an architecture definition. It says nothing about what Python version
loaded it, what version of PyTorch or scikit-learn the inference code
expects, what system libraries those packages link against, or what
hardware drivers need to be present. All of that context lives implicitly
in whatever machine the data scientist happened to train on. The moment
that model needs to run somewhere else — a teammate's laptop, a CI runner,
a production cluster — every one of those implicit assumptions becomes a
place reproducibility can quietly break. This is the exact problem
containerization exists to solve, and it's worth being precise about the
mechanism rather than treating "just use Docker" as a slogan.

A container image is a filesystem snapshot plus a declared set of
instructions to produce it, built from a Dockerfile. The image you build
from `python:3.11-slim`, `pip install -r requirements.txt`, and `COPY`
your model-serving code into place is a single artifact that contains the
exact interpreter, the exact library versions pinned in that requirements
file, and the model code itself. When that image runs anywhere Docker
runs — your laptop, a GitHub Actions runner, a Kubernetes node — it runs
against that same filesystem, not against whatever happens to already be
installed on the host. The host's Python version, its globally installed
packages, its OS patches: none of it matters, because the container never
touches them. That isolation is the whole mechanism, and it's what turns
"it works on my machine" from an excuse into a bug report, because the
machine's state is no longer part of the equation.

## Why layer order is not cosmetic

Docker builds an image as a stack of layers, one per instruction, and
caches each layer by hashing its instruction plus its inputs. On a
rebuild, Docker reuses every cached layer up to the first one whose inputs
changed, and rebuilds everything after that from scratch. This has a
direct, practical consequence for how you write a Dockerfile for a model
service: put the things that change rarely — installing the base OS
packages, installing your pinned dependencies from a lockfile — near the
top, and put the things that change on every commit — your application
code, and especially your model weights — near the bottom.

Get this backwards (say, copying your entire project directory, weights
included, before running `pip install`) and every commit invalidates the
dependency-install layer too, so every CI build reinstalls every package
from scratch even though nothing about the environment changed. For a
model image where dependencies can include multi-gigabyte GPU libraries,
that's the difference between a two-minute CI build and a twenty-minute
one, repeated on every push. This isn't a micro-optimization; it's the
difference between a packaging step your team tolerates and one it works
around by skipping CI.

## Two artifacts, two lifecycles

It's tempting to think of "the model" and "the container" as the same
thing, but keeping them distinct is what makes deployments safe. A model
registry (the prerequisite to this lesson) versions trained weights and
their evaluation metadata — this is model version `v12`, trained on this
data, scoring this well. A container image versions the runtime those
weights execute inside — this is service version `2.4.1`, running
PyTorch `2.3`, CUDA `12.1`. A production deployment pins both: this
container image, loading that specific registry version at startup or
build time. If you only version one of the two, you lose the ability to
answer a question that comes up the moment something breaks in
production: did the weights change, did the runtime change, or both? A
build that bakes an immutable model version into a tagged image, rather
than pulling "latest" from the registry at container start, makes every
deployed instance of that image byte-for-byte identical and lets a
rollback be as simple as redeploying the previous image tag.

## Why this is the foundation for everything after it

Every later stage in this group, and in the CI/CD and serving groups that
follow it, assumes the model is already packaged as a container. A
pipeline can't run automated tests against "a model" — it runs tests
against a built image. A deployment strategy can't gradually shift traffic
between "old model" and "new model" — it shifts traffic between two
running container versions. Getting containerization right here, with
deliberate layering and a clean separation between model version and
runtime version, is what makes every later promise about reproducible,
automatable delivery actually true instead of aspirational.
