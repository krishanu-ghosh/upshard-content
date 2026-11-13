---
title: API and infrastructure security for model serving
description: ShadowRay and the TorchServe exposure show that ML-serving infrastructure gets ordinary infrastructure bugs when it skips standard API hardening.
roadmap: ai-red-teaming
node: api-and-infrastructure-security
group: system-attacks
video:
  provider: youtube
  id: "tQEkqgOMA8Q"
  start: 0
  end: null
creator:
  name: AppSec Village
  channel_url: https://www.youtube.com/@AppSecVillage
duration_seconds: 2400
level: intermediate
prerequisites: [insecure-deserialization-and-rce]
takeaways:
  - The software that serves and orchestrates a model — the inference server, the job scheduler, the vector database, the metrics dashboard — is ordinary infrastructure that inherits ordinary infrastructure vulnerabilities.
  - "ShadowRay showed that Ray's Jobs API shipped reachable with no authentication by default, and attackers used that exposure in the wild to hijack GPU compute at scale."
  - A dashboard or management port designed for a trusted internal network becomes a direct remote-code-execution path the moment it is reachable from the public internet, regardless of how it is labeled.
  - Calling something "just an ML tool" does not exempt it from authentication, network segmentation, and rate limiting — the same baseline every other production service is held to.
check_questions:
  - q: What was the actual root cause behind the ShadowRay incidents?
    a: Ray's dashboard and Jobs API were reachable without any authentication by default, so anyone who could route to that port could submit arbitrary jobs to the cluster, and attackers exploited internet-exposed instances in the wild to hijack GPU compute for things like cryptomining.
  - q: Why does an ML-serving framework's management or dashboard port deserve the same scrutiny as a production API endpoint?
    a: Because it is functionally the same kind of surface — an interface that accepts requests and acts on them — and exposing it to an untrusted network turns whatever that interface can do, from reading internal state to scheduling arbitrary jobs, into something anyone on the internet can trigger.
  - q: What two concrete hardening steps would have blunted both the ShadowRay and TorchServe incidents?
    a: Requiring authentication on management and dashboard endpoints before they accept any request, and keeping those ports off the public internet entirely by binding them to an internal network or putting them behind a firewall or VPN.
deeper: []
related_formulas: []
---

## The infrastructure around the model is still infrastructure

It's easy to focus all of an AI red-teaming mental model on the LLM itself —
prompts, jailbreaks, hallucinated outputs — and forget that a deployed model
sits inside an ordinary software stack: an inference server accepting
requests, an orchestration framework scheduling work across GPUs, a vector
database answering similarity queries for a RAG pipeline, a dashboard
showing cluster health. None of that surrounding machinery is magic just
because the payload it eventually produces is a language model's output.
It's a set of network services, each with an API, a listening port, and an
authentication model — or, in two well-documented real incidents, the
absence of one.

## ShadowRay: an unauthenticated API in the wild

Ray is a popular distributed-computing framework widely used to orchestrate
large-scale ML training and serving jobs. Its dashboard exposes a Jobs API
that lets a caller submit arbitrary code to run across the cluster — a
deliberate and useful feature for a trusted operator. The problem, documented
under the name ShadowRay, is that this API shipped reachable with no
authentication enabled by default, and a meaningful number of organizations
ran it exposed directly to the public internet without realizing the
dashboard port carried that capability. The consequence wasn't theoretical:
security researchers and attackers alike found these open dashboards and used
the Jobs API exactly as designed — to submit jobs — except the jobs they
submitted hijacked expensive GPU compute for cryptomining and other abuse, on
infrastructure the operator believed was internal. A related exposure hit
TorchServe, where the management API used to register and configure deployed
models was likewise reachable without authentication in default
configurations, giving an attacker a path to load or replace the model a
production endpoint was actually serving.

## Why this keeps happening in ML stacks specifically

Neither incident involved a novel attack technique. An unauthenticated
management API that accepts arbitrary commands is a decades-old category of
bug; the interesting part is why it recurred here. ML infrastructure tends to
be adopted fast, by teams optimizing for getting a training run or a serving
endpoint working today, often by engineers whose expertise is modeling, not
network security. Frameworks built for the assumption of a trusted cluster
network — the normal case in a research lab or a tightly controlled data
center — get lifted directly into cloud deployments where "internal" no
longer means "unreachable," because a misconfigured security group or a
default-open port puts that same interface on the public internet. The
software did exactly what it was designed to do; what changed was who could
reach it.

## Generalizing the lesson

The specific fix for ShadowRay or the TorchServe exposure is narrow — patch,
enable authentication, close the port — but the principle it illustrates is
broad and durable: every piece of software sitting between the public
internet and a model, whether it's the inference endpoint itself, the
orchestration layer behind it, the vector store backing a RAG pipeline, or a
metrics and admin dashboard, needs the same baseline hardening any other
production API gets. That means authenticating every endpoint that accepts a
request, never exposing a dashboard, admin console, or management port
directly to an untrusted network, applying rate limiting to inference
endpoints so they can't be trivially abused for resource exhaustion or
scraping, and treating default configurations as insecure until verified
otherwise, because defaults in fast-moving ML tooling are frequently tuned
for convenience during development rather than for a hostile network. "It's
just an ML tool" has never been a reason an attacker respects, and in both of
these real, publicly documented cases, it wasn't a reason the infrastructure
respected either — it was simply a port left open.
