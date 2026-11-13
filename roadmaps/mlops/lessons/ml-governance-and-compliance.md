---
title: ML governance and compliance
description: Model cards, approval workflows, and audit trails — the documentation and process layer that makes a production model accountable.
roadmap: mlops
node: ml-governance-and-compliance
group: monitoring-governance
video:
  provider: youtube
  id: "Q020C-Jw0o8"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 551
level: advanced
prerequisites: [data-and-concept-drift, explainability-and-interpretability]
takeaways:
  - A model card is a short, standardized document attached to a model that states what it was trained on, how it performs across relevant subgroups, its known limitations, and its intended (and explicitly unintended) use cases, so anyone deploying or auditing it doesn't have to reverse-engineer that from the code.
  - An approval workflow is a required, logged human checkpoint before a model change reaches production — someone with the authority to say no reviews evaluation results, fairness metrics, and documentation before promotion, and that review itself becomes part of the audit trail.
  - Regulatory requirements like explainability mandates and bias audits aren't separate bureaucratic add-ons; they're satisfiable only because of groundwork laid earlier in this group — drift detection and SHAP/LIME explanations are what actually produce the evidence a governance process needs to review.
check_questions:
  - q: What does a model card contain, and why does it exist?
    a: A standardized summary of a model's training data, measured performance across relevant subgroups, known limitations, and intended versus unintended uses, so that anyone evaluating or deploying the model has that context without having to dig through code or ask the original author.
  - q: What is an approval workflow in the context of ML governance?
    a: A required, logged checkpoint where a person with the authority to block a release reviews a model's evaluation results and documentation before it's promoted to production, creating an auditable record of who approved what and why.
  - q: Why can't a bias audit be done well without the explainability techniques covered in the previous lesson?
    a: Because a meaningful bias audit needs to see which features are driving predictions for different subgroups, not just compare overall accuracy numbers, and that per-prediction visibility is exactly what SHAP and LIME are built to provide.
deeper:
  - label: "Responsible AI vs AI Governance Explained | What's the Difference? — School of Responsible AI (SoRAI)"
    url: "https://www.youtube.com/watch?v=zrToOR1fBeI"
related_formulas: []
---

## Governance is what makes "it works" accountable

Every lesson before this one in the roadmap answers some version of "does
the model work, and does it keep working." **Governance** answers a
different question: can anyone — a regulator, an auditor, a future
teammate, the person the model made a decision about — find out *why* it
was allowed into production, *who* signed off on it, and *what happens*
when something goes wrong? A model that performs well but has no
documentation, no approval trail, and no way to explain an individual
decision is a liability regardless of its accuracy, because the moment it
causes harm or gets challenged, there's nothing to point to.

## Model cards: the document that travels with the model

A **model card** is a short, structured document that ships alongside a
trained model, originally formalized by a 2019 Google research paper and
now a near-standard practice across the industry. A useful model card
typically states: what data the model was trained on, including its
provenance and known gaps; the metrics used to evaluate it, broken out
across relevant demographic or operational subgroups rather than just one
aggregate number (a model that's 95% accurate overall but 70% accurate for
a specific subgroup has a very different risk profile than the aggregate
number suggests); known limitations and failure modes; and, critically,
the intended use cases the model was built and validated for, alongside
explicitly out-of-scope uses it should never be deployed for. A fraud
model's card stating "not validated for use in credit decisions" is the
difference between a documented limitation and a costly surprise six
months later when someone repurposes it.

Model cards aren't meant to be exhaustive engineering documentation — the
model registry from earlier in this roadmap already holds versioning and
lineage. A model card is meant to be readable by someone who isn't the
model's author: a reviewer, a downstream team deciding whether to
integrate it, a regulator asking what the model was validated for.

## Approval workflows: a required human checkpoint

Continuous delivery for models, covered earlier in this roadmap,
automates getting a validated model to production quickly. Governance
adds a deliberate point of friction back in: before a model (or a
significant change to one) reaches production, a person with real
authority to block the release reviews its evaluation results, its
fairness metrics across subgroups, and its model card, and records that
decision. This isn't bureaucracy for its own sake — it's the mechanism
that prevents "the pipeline said the tests passed" from being the only
checkpoint a consequential model ever passes through, and it produces an
auditable record: who approved this version, when, based on what evidence.
When something goes wrong later, that record is what lets a team
reconstruct whether the failure was a gap in the process or a genuine
surprise.

## Where regulation turns these from best practice into requirement

Regulatory frameworks emerging across finance, healthcare, hiring, and
general-purpose AI increasingly codify exactly this: a right to an
explanation for an automated decision that affects someone materially, a
requirement to audit a model for disparate impact across protected
groups before and after deployment, and a requirement to document a
model's intended use. None of these are satisfiable after the fact by
writing a report — they require the explainability tooling from the
previous lesson to actually produce per-decision reasoning, and the drift
monitoring from two lessons ago to show the model's behavior has stayed
within the bounds it was audited for. Governance is the discipline that
turns those earlier technical capabilities into an answerable record, and
a bias audit without underlying attribution is just a plausible-sounding
accuracy table.

## Carrying this into the final lesson

Good governance also determines how an organization responds when a
model does fail in production — a documented approval trail and a clear
owner are exactly what the incident response process in the next lesson
needs to move fast instead of spending the first hour figuring out who
has the authority to roll a model back.
