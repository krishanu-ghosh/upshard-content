---
title: Infrastructure as code
description: Why declaring ML infrastructure in version-controlled files, instead of clicking around a cloud console, is the difference between reproducible and lost.
roadmap: mlops
node: infrastructure-as-code
group: orchestration-infra
video:
  provider: youtube
  id: "l5k1ai_GBDE"
  start: 0
  end: null
creator:
  name: TechWorld with Nana
  channel_url: https://www.youtube.com/@TechWorldwithNana
duration_seconds: 1095
level: intermediate
prerequisites: [workflow-orchestration]
takeaways:
  - Infrastructure as code (IaC) means infrastructure is defined in version-controlled, declarative files that a tool reconciles against reality, rather than built by hand through a console.
  - A declarative IaC tool computes a diff between the desired state you wrote and the actual state of the cloud, and changes only what differs — it doesn't blindly rerun every step.
  - For ML specifically, IaC is what makes expensive, easy-to-misconfigure resources like GPU clusters reproducible, reviewable, and safely disposable instead of a one-off someone has to remember how to rebuild.
check_questions:
  - q: What does it mean for an IaC tool to be declarative rather than imperative?
    a: You describe the desired end state of the infrastructure (e.g., "three GPU nodes of this type exist") and the tool figures out what actions get you there, rather than you scripting the exact sequence of API calls yourself.
  - q: Why is manually provisioned ML infrastructure a bigger liability than manually provisioned infrastructure in general?
    a: ML infrastructure (GPU clusters, large storage volumes, specialized networking) is expensive to leave running, slow and error-prone to recreate from memory, and often provisioned under time pressure to unblock a training run, making undocumented manual setups both costly and hard to reproduce.
  - q: What does an IaC tool's "plan" step actually compute?
    a: The diff between the infrastructure described in the code and the infrastructure that currently exists in the cloud account, so you can review exactly what will be created, changed, or destroyed before anything happens.
deeper: []
related_formulas: []
---

## The console is where reproducibility goes to die

Clicking through a cloud provider's console to spin up a GPU training
cluster works, and it works *today*, for *you*, because you remember what
you clicked. Three months later, when that cluster needs to be rebuilt
after an accident, scaled up for a bigger model, or replicated in a second
region for redundancy, that knowledge is gone unless you wrote it down
exactly — and "exactly" is a high bar for a sequence of console clicks
across a dozen screens. Infrastructure as code solves this by making the
infrastructure definition itself a file: version-controlled, reviewable in
a pull request, and re-runnable by a tool rather than a memory. The
infrastructure stops being "whatever currently exists in the account" and
becomes "whatever this file says it should be" — a meaningfully different
and far more durable kind of truth.

## Declarative, not a shell script

The distinguishing feature of tools like Terraform, Pulumi, or Google's
Deployment Manager is that they're declarative: you write what you want
the end state to look like (a GPU node pool with this instance type and
this count, a storage bucket with this retention policy, a network with
this topology), not the sequence of commands to get there. The tool
compares your declared state against the actual state of the cloud account
and computes a plan — a diff of exactly what will be created, modified, or
destroyed. This matters enormously in practice: if nine of your ten
described resources already exist, the tool only touches the tenth. An
imperative script, by contrast, either has to be re-run in full (often
failing on the parts that already exist) or needs its own hand-written
logic to check what's already there — logic that itself can drift out of
sync with reality.

## Why ML infrastructure specifically needs this discipline

General-purpose infrastructure (a web server, a database) benefits from
IaC, but ML infrastructure makes the case sharper for three reasons. First,
cost: a GPU cluster idling because nobody remembered to tear it down after
a training run can cost more per day than the entire rest of a team's cloud
bill, and IaC makes "tear it down" a one-command, auditable action instead
of a scavenger hunt through console tabs. Second, reproducibility of the
*environment*, not just the model: a training run's result depends on
driver versions, interconnect topology, and instance type, and if that
setup lived only in one engineer's click history, nobody else can
faithfully reproduce the conditions that produced a given model. Third,
blast radius: ML infrastructure is frequently provisioned under time
pressure ("I need this cluster for tomorrow's run") which is exactly the
condition under which people skip documentation — IaC removes the tradeoff
between speed and leaving a record, because the record *is* the fast path
once the first version exists.

## State, drift, and the trap that catches teams later

IaC tools track a state file recording what they believe exists. Problems
show up when reality and that state file disagree — someone manually
deletes a resource, or modifies it in the console "just to test something."
This is called drift, and it's the main operational hazard of adopting IaC:
the tool's next plan may propose destroying and recreating a resource it
thinks is wrong, when actually its own state is stale. The discipline this
demands is strict: once a piece of infrastructure is managed by code, all
changes to it go through the code, not the console, even for a quick fix.
Teams that treat this as a strong rule rather than a suggestion get the
full benefit; teams that treat IaC as documentation-after-the-fact while
still clicking around for real changes end up with state files nobody
trusts, which is worse than not having IaC at all because it creates false
confidence.

## Why this matters going forward

Treating infrastructure as code is what makes the rest of an ML platform
buildable by more than one person and recoverable after things go wrong.
It's also the foundation the next lesson builds on directly: managed cloud
ML platforms are themselves just infrastructure, and deciding whether to
provision your own training infrastructure or hand that job to a managed
service is a much easier call to make well when your current setup is
already declared, versioned, and reproducible rather than tribal knowledge
sitting in one person's head.
