---
title: Continuous monitoring and detection
description: Red teaming and guardrails happen before launch; production monitoring is how you catch drift and live attacks after launch.
roadmap: ai-red-teaming
node: continuous-monitoring-and-detection
group: defense
video:
  provider: youtube
  id: "kH4ZoZSvddM"
  start: 0
  end: null
creator:
  name: DeepLearningAI
  channel_url: https://www.youtube.com/@Deeplearningai
duration_seconds: 3120
level: intermediate
prerequisites: [llm-security-testing-tools, adversarial-training-and-robust-design]
takeaways:
  - Red teaming and guardrails are checks run before and around launch, but attackers keep probing a live system indefinitely, so production needs its own detection layer that runs continuously.
  - Live attack detection means logging and flagging suspicious prompts and responses in real time, the same way a security team watches for anomalous traffic on any other production system.
  - Drift detection borrows general ML-monitoring techniques — watching how the input distribution and model behavior change over time — and repurposes them for security, because a model that quietly drifts away from its red-teamed baseline may be more vulnerable than the version that was originally tested.
  - None of this works without operational observability — tracing, logging, and alerting wired deeply enough into the system that a detected anomaly can actually be traced back to a specific request and acted on.
check_questions:
  - q: Why isn't red teaming before launch sufficient on its own to keep an LLM system safe?
    a: Red teaming and guardrails test a snapshot of the system before or at launch, but attackers keep probing after launch indefinitely and the system itself can change over time, so without continuous monitoring there is no way to know whether the system is still as safe as it was when it was tested.
  - q: What's the difference between live attack detection and drift detection?
    a: Live attack detection flags suspicious individual prompts or responses as they happen, in real time, while drift detection watches for the input distribution or the model's behavior gradually changing over a longer period, which can erode safety margins even without any single malicious-looking request.
  - q: Why is operational observability described as a prerequisite for the other two monitoring facets, not a separate concern?
    a: Detecting an attack or a drift signal is only useful if it can be traced back to specific requests, model versions, and context so a team can actually investigate and respond, and that traceability is exactly what tracing and logging infrastructure provides.
deeper:
  - label: "Model Monitoring and Drift Detection with Evidently AI — AIOps Academy"
    url: https://www.youtube.com/watch?v=ZJj042dNWE0
  - label: "Datadog LLM Observability: Monitor and secure your AI workloads — Datadog"
    url: https://www.youtube.com/watch?v=uA4pBu8leyU
related_formulas: []
---

## Red teaming finds what's wrong once; monitoring finds what's wrong continuously

Every lesson before this one in the `defense` group describes something you do to a system before or around the moment it ships: red-team it, wrap it in guardrails, train resistance into the model itself. All of that work answers one question — is this system safe as of today's testing? It says nothing about tomorrow.

An attacker doesn't stop probing once a product launches. A model can get silently swapped for a newer version behind the same API. Usage patterns shift in ways nobody anticipated during testing. A guardrail tuned against last quarter's known jailbreaks has no special insight into next quarter's. Continuous monitoring treats "is this system still safe" as an ongoing question, re-asked constantly in production, rather than a box checked once before launch.

## Live detection: watching for attacks as they happen

The most direct form of this is real-time detection of attack and misuse attempts against a deployed system: logging prompts and responses, running them (or samples of them) through classifiers similar to the guardrails already covered, and flagging patterns that look like injection attempts, jailbreak attempts, or policy-violating output as they occur, rather than after the fact.

This is conceptually close to intrusion detection on a conventional network. You're not trying to make the attack impossible — you're trying to notice it fast enough to respond: rate-limit an abusive account, roll back a bad prompt-engineering change, or feed the captured attempt back into the next round of red-teaming and adversarial training as a new example to defend against. The loop matters here. Production detections are some of the highest-quality red-team data available, because they're real attacks that actually worked against the real system, not simulated ones.

## Drift detection: when the baseline itself moves

Drift detection is a subtler, slower-moving problem. It's worth noting explicitly that it isn't a security-specific technique at all — it's standard ML-operations practice, the kind of thing a tool like Evidently AI was built for generally, across any deployed model, repurposed here for a security end.

A model's input distribution changes as user behavior changes. A fine-tuning update, a new system prompt, or an upstream dependency change can shift the model's output behavior in ways nobody deliberately tested for. None of that needs to look like an attack to be dangerous — a model that has quietly drifted away from the behavior your red-team report actually evaluated may now have safety margins thinner than anyone realizes, with no single incident to point to. Monitoring for drift means tracking statistical properties of inputs and outputs over time and alerting when they move meaningfully away from the baseline established during testing, so an erosion in safety margin gets caught before an attacker finds it.

## Observability: the infrastructure that makes detection actionable

Neither live detection nor drift detection means anything without the unglamorous infrastructure underneath both: tracing requests through a system, logging enough context to reconstruct what happened, and alerting routed to someone who can act. This is the layer tools like Datadog's LLM observability features target — not a new detection algorithm, but the plumbing that makes an anomaly traceable back to a specific request, a specific model version, a specific user, and a specific point in a multi-step agent trace, at the scale a production system actually runs at. Without it, a flagged anomaly is just a number on a dashboard nobody can connect back to a cause.

The throughline across this whole lesson is one sentence worth keeping: red teaming finds what's wrong before launch, monitoring is how you find out it's still — or newly — wrong after launch. In production the adversary isn't a fixed test case. It's a continuing process that doesn't stop when the engagement report gets filed.
