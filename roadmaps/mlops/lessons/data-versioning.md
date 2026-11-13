---
title: Data versioning
description: Tracking dataset changes the way Git tracks code, so a training run can be reproduced against the exact data it used.
roadmap: mlops
node: data-versioning
group: data-foundations
video:
  provider: youtube
  id: "UbL7VUpv1Bs"
  start: 0
  end: null
creator:
  name: DVCorg
  channel_url: https://www.youtube.com/@dvcorg8370
duration_seconds: 302
level: beginner
prerequisites: [reproducibility-and-the-training-serving-gap]
takeaways:
  - Git tracks code well but handles large binary files badly, which is why datasets need a separate, parallel versioning mechanism rather than being committed directly.
  - A data version is best stored as a content hash plus a small pointer file, so Git can track "which version of the data" without ever storing the data itself.
  - Without data versioning, "which data trained this model" becomes a question nobody can answer precisely once the dataset has been cleaned, re-pulled, or edited even once.
check_questions:
  - q: Why can't you just commit a dataset into Git the same way you commit code?
    a: Git is built to diff and store text efficiently, and it chokes on large binary or tabular files, bloating the repository and making every clone and checkout slow.
  - q: What does a data versioning tool actually store in Git, if not the data itself?
    a: A small pointer file containing a content hash and metadata that identifies exactly which version of the dataset, stored separately in object storage, corresponds to a given commit.
  - q: What question does data versioning let you answer that you otherwise couldn't?
    a: Exactly which version of a dataset produced a given trained model, so the training run can be reproduced or audited later.
deeper:
  - label: "Master MLOps: Getting Started with Data Version Control (DVC) — MLWorks"
    url: "https://www.youtube.com/watch?v=D_Ha_bNviHk"
related_formulas: []
---

## The problem Git doesn't solve

Every ML team eventually hits the same wall: code is versioned, but data
isn't. You can check out any commit from three months ago and see exactly
what the training script looked like, but you have no equally precise way
to answer "what did the training data look like at that point?" Someone
re-ran a cleaning script and it dropped a few thousand duplicate rows.
Someone else pulled a fresh export from the production database that
included a week of new records. A label got fixed. None of this is
recorded anywhere a code diff would show it, because the dataset itself —
often a multi-gigabyte CSV, Parquet file, or directory of images — was
never something anyone thought to put under the same discipline as code.
The result is a model nobody can fully explain six weeks later: the code
that trained it is right there, but the data it trained on has silently
moved on.

You might reasonably ask why not just commit the dataset to Git like
everything else. The answer is mechanical: Git was built around diffing
and storing text efficiently, computing line-by-line deltas between
versions of a file. A 2 GB Parquet file or a folder of ten thousand
images doesn't diff that way — Git ends up storing something close to a
full new copy on every change, and the repository balloons, cloning slows
to a crawl, and every teammate's laptop starts choking on a `git pull`.
Data needs version control with the same properties Git gives code —
history, diffing, branching, the ability to go back — without actually
living inside Git's object store.

## How data versioning actually works

The pattern used by tools built for this (DVC is the best-known, but the
idea generalizes) is to split the problem in two. The dataset itself lives
in cheap bulk storage — a local cache, S3, GCS, or similar — identified by
a content hash: a fingerprint computed from the file's actual bytes, so
two files with identical content get the identical hash regardless of
filename or location. What goes into Git instead is a tiny pointer file,
often just a few lines of YAML, containing that hash and some metadata
like file size. Git tracks the pointer file the way it tracks any small
text file — cheaply, with full diff and history — while the heavy data sits
outside the repo entirely.

This gives you the property that actually matters: checking out a Git
commit also tells you, unambiguously, which exact dataset version belongs
with that code. If the pointer file at commit `a3f9c1` has hash
`e4d909c2...`, that hash can be resolved back to the exact bytes of the
dataset used to train whatever model that commit produced — not "the
dataset as it looks today," not "approximately the same data," but the
literal same bytes, byte for byte. Change the data at all — fix one label,
drop one row — and the hash changes, a new pointer gets committed, and the
old version remains retrievable forever through Git history, exactly like
an old version of a source file.

## What this buys you in practice

Picture a model that performs noticeably worse after a retrain. Without
data versioning, debugging this means guessing: was it the new
hyperparameters, a code change, or did the data shift? With data
versioning, you can check out the exact pointer commit from the last known
good run, pull that exact dataset back, and retrain against it to isolate
whether the regression is a data problem or a code problem — a question
that's otherwise nearly impossible to answer cleanly after the fact.
It also makes rollback real: if a bad data export got merged into the
training set, you revert the pointer file the same way you'd revert a
buggy code change, and the bulk storage already has the previous version
sitting there, ready to pull.

## Why this is the foundation, not a nice-to-have

Data versioning is the piece that makes every later practice in this
group possible to build on. Reliable pipelines, feature stores, and
lineage tracking all assume you can refer to "this dataset" as a fixed,
addressable thing rather than a moving target that happens to live at a
file path. Skip this step and every later claim about reproducibility —
"we can retrain this model exactly" — is actually just a hope, resting on
nobody having touched the data since. Get it right here, and everything
built on top of it, from pipelines to audits, has solid ground to stand
on.
