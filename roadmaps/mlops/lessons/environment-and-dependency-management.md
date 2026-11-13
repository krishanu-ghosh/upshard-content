---
title: Environment and dependency management
description: Pinning library versions with lockfiles and matching CUDA driver versions, the two halves of making training and serving environments actually match.
roadmap: mlops
node: environment-and-dependency-management
group: packaging
video:
  provider: youtube
  id: "0f3moPe_bhk"
  start: 0
  end: null
creator:
  name: ArjanCodes
  channel_url: https://www.youtube.com/@ArjanCodes
duration_seconds: 882
level: intermediate
prerequisites: [containerizing-models]
takeaways:
  - A requirements file that lists loose version ranges resolves differently on different days, while a lockfile freezes every package, including transitive dependencies, to one exact resolved set.
  - Pinning your own Python libraries doesn't touch the hardware layer, and a CUDA toolkit version that's incompatible with the installed GPU driver fails or silently miscomputes independently of anything your lockfile controls.
  - A model that works in training but breaks in serving is usually not a code bug; it's an environment that was never actually pinned, just assumed to be stable.
check_questions:
  - q: Why does a requirements.txt with loose version ranges fail to guarantee reproducibility, even if it never changes?
    a: Because its ranges still allow pip to resolve to a different compatible version of each package (and of each package's own dependencies) depending on what's available and when the install runs, so the same file can produce different installed sets on different days.
  - q: What's the difference between what a lockfile pins and what a CUDA driver version controls?
    a: A lockfile pins the exact Python package versions your application code depends on, while the CUDA driver is a separate, GPU-level component that a lockfile has no mechanism to pin or install, and a mismatch there breaks GPU code regardless of how well your Python dependencies are pinned.
  - q: Why can a model that trains correctly still fail or misbehave when moved to a serving environment?
    a: Because the serving environment can have a different resolved set of library versions, a different CUDA toolkit version, or a different GPU driver than training used, and any of those mismatches can change numerical behavior or cause an outright failure that never appeared during training.
deeper:
  - label: "CUDA Tutorials I CUDA Compatibility — NVIDIA Developer"
    url: "https://www.youtube.com/watch?v=H3AQnlpxk0c"
related_formulas: []
---

## Two separate pinning problems, both called "dependencies"

"Environment and dependency management" sounds like one problem, but it's
actually two, and they fail in different ways. The first is library
pinning: making sure the exact versions of every Python package your code
imports are the same in training and in serving. The second is hardware
and driver compatibility: making sure the GPU-level software stack
underneath those libraries — the CUDA toolkit and the GPU driver — is
consistent too. A lockfile fixes the first problem completely and the
second problem not at all. Treating them as one problem is exactly how
teams end up confident their dependencies are "pinned" while still hitting
environment-only failures in production.

## Why a requirements file isn't a lock

A typical `requirements.txt` with entries like `torch>=2.0` or even
`torch==2.3.0` looks pinned, but it usually isn't, for a subtle reason:
your code doesn't only depend on `torch`, it depends on everything `torch`
depends on, and everything those packages depend on, recursively. Unless
every single one of those transitive dependencies is also pinned to an
exact version, `pip install` is free to resolve a different compatible
set each time it runs — a new release of a sub-dependency published since
your last install, a different resolution order, a different platform
wheel. Two installs run weeks apart from the *same* `requirements.txt` can
produce different installed packages.

A lockfile (what tools like Poetry, pip-tools, or uv produce) solves this
by recording the full resolved dependency graph — package and version for
every direct and transitive dependency — as a single artifact that's
regenerated deliberately, not implicitly on every install. Installing from
a lockfile doesn't run a fresh resolution at all; it installs exactly the
versions already written down. This is the difference between "my code
should work with roughly these libraries" and "my code runs against this
exact, named set of bytes," and it's what makes a training run from six
months ago actually reproducible today instead of merely plausible.

## The layer a lockfile can't touch: CUDA

Here's where the second half of this lesson matters, and where loose
mental models of "dependencies" go wrong. GPU-accelerated libraries like
PyTorch or TensorFlow are compiled against a specific version of the CUDA
toolkit, and that toolkit in turn requires a minimum version of the
*GPU driver* installed on the host machine — a kernel-level component that
lives outside Python entirely and that no `pip install` or lockfile can
manage. The driver is typically installed once by whoever provisions the
machine or the base container image, and it's backward compatible with
older CUDA toolkit versions but not with arbitrarily newer ones.

This produces a specific, very common failure: a team trains on a machine
with a recent driver and a correspondingly recent CUDA toolkit, builds a
serving image using a `torch` wheel compiled against that same toolkit
version, and deploys it to production infrastructure whose GPU driver is
older — maybe because that fleet hasn't been re-provisioned recently.
Depending on the exact mismatch, this surfaces as the process refusing to
start with a CUDA initialization error, or worse, silently falling back to
CPU execution with no error at all, turning a GPU-serving deployment into
one that technically runs but is quietly far slower than expected. None of
this shows up by auditing `requirements.txt` — the Python versions can be
identical on both machines. The fix is treating the driver version as
a dependency to be checked and matched explicitly, the same way you'd
check a library version, rather than an invisible property of whatever
machine happens to be running the container.

## Pinning both halves, deliberately

The practical discipline this lesson is pointing at is: pin your Python
dependency graph with a real lockfile, commit that lockfile alongside your
code, and regenerate it only as a deliberate action you can review — and
separately, pin and document the CUDA toolkit and minimum driver version
your container image requires, verifying the serving fleet's driver
actually meets it before deploying. Container base images (like
`nvidia/cuda:12.1-runtime`) encode the first of these choices explicitly,
which is why building your serving image from a CUDA-tagged base rather
than a generic Python base is part of making this reproducible rather than
assumed.

## Why this is where "it works in training" goes to die

Of every category of training-to-serving failure, environment mismatch is
the most common precisely because it's invisible until it isn't — the
model code hasn't changed, the data hasn't changed, and yet behavior or
performance has. Pinning dependencies fully, at both the library layer and
the hardware-driver layer, is what converts "environment" from a hidden
variable into a documented, checkable one — which is exactly the
discipline the next lesson builds on when it looks at portable model
formats that reduce how much of this environment a served model needs to
carry with it at all.
