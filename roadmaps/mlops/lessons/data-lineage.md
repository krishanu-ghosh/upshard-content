---
title: Data lineage
description: Tracing where a piece of data or feature came from and what transformed it — essential for debugging predictions and for audit and compliance.
roadmap: mlops
node: data-lineage
group: data-foundations
video:
  provider: youtube
  id: "whF6dH-XZUM"
  start: 0
  end: null
creator:
  name: The Data and AI Guy
  channel_url: https://www.youtube.com/@thedataandaiguy
duration_seconds: 735
level: beginner
prerequisites: [data-pipelines-for-ml]
takeaways:
  - Data lineage is a traceable record of where a piece of data originated and every transformation it passed through on the way to being used by a model.
  - Without lineage, debugging a bad prediction means manually reconstructing a pipeline's history from memory and scattered logs, which doesn't scale past a handful of pipelines.
  - Lineage is also a compliance requirement in regulated settings, where you must be able to show exactly which source data and transformations produced a given model decision.
check_questions:
  - q: What does a data lineage record actually capture?
    a: The origin of a piece of data and the full chain of transformations, joins, and pipeline steps it passed through before reaching its current form.
  - q: Why does debugging "why did this prediction change" get hard without lineage, even if your pipelines are reliable?
    a: Because without a recorded trail you have to manually reconstruct which upstream source, transformation, or pipeline version produced a given feature value, which becomes impractical once there are more than a few pipelines and feeds involved.
  - q: Besides debugging, what's a second major reason organizations invest in data lineage?
    a: Regulatory and audit requirements, since some industries require being able to show exactly which data sources and transformations led to a specific automated decision.
deeper:
  - label: "Data Lineage: An Origin Story — SAS Software"
    url: "https://www.youtube.com/watch?v=xIQG6A21oGw"
related_formulas: []
---

## What lineage actually tracks

Data lineage is the recorded trail connecting a piece of data at any point
in a system back to where it originated and through every transformation
it passed through to get there. Concretely, that means: this feature
value came from this column, in this table, which was populated by this
join, which pulled from this source table, which was last updated by this
pipeline run, at this timestamp, using this version of the transformation
code. It sounds like bookkeeping, and in a sense it is — but it's
bookkeeping that becomes indispensable the moment something goes wrong and
you need to know exactly what fed into a specific result, rather than
roughly what you believe fed into it.

The distinction worth being precise about is lineage versus versioning,
covered earlier in this group. Versioning answers "what did this dataset
look like at a point in time" — it's a snapshot capability. Lineage
answers a different question: "what is the chain of transformations and
sources that produced this specific value, here, now" — it's a graph of
dependencies, not a snapshot. You typically need both: versioning to
reproduce a past state, lineage to understand how any given state was
derived.

## A concrete debugging scenario

Imagine a fraud-detection model starts flagging a noticeably higher
fraction of legitimate transactions as fraudulent, starting on a specific
date. Without lineage, the investigation is detective work: someone has
to remember or reconstruct which pipelines feed this model's features,
check each one's logs by hand, compare recent code changes across
multiple repositories, and hope the actual cause — say, an upstream
"transaction location" field that started reporting in a different
coordinate format after a vendor API update — surfaces before the
on-call engineer runs out of patience or the business runs out of
tolerance for false declines. With lineage recorded, the same
investigation starts from the affected feature and walks backward through
its recorded dependency graph directly to the exact upstream table and
the exact pipeline run that changed, narrowing a search that could take
days down to something closer to minutes. This is the practical payoff:
lineage turns "why did this change" from an archaeology project into a
graph traversal.

## Lineage at different granularities

Lineage can be captured at different levels of detail, and the right
level depends on what you need to answer. Table-level lineage records
which tables feed which other tables — coarse, but enough to answer "if I
change this source table, what downstream models are affected." Column-
level lineage goes further, tracking which specific source columns feed
which specific derived columns or features — necessary when you need to
answer "if this one field's meaning changes, which exact features and
models does that touch," which table-level lineage can't answer
precisely. Row-level or record-level lineage, the finest grain, tracks
individual records through transformations and is typically reserved for
cases with strict audit requirements, since it's the most expensive to
capture and store at scale. Most teams start with table-level lineage
because it's cheap to automate — many pipeline and orchestration tools
can emit it as a byproduct of running — and add finer granularity only
where the debugging or compliance need actually justifies the cost.

## Why regulated industries treat this as mandatory, not optional

Beyond debugging, lineage is often a hard legal requirement. In finance,
healthcare, and other regulated sectors, a model that materially affects
a person — denying a loan, flagging a claim, influencing a diagnosis — can
be subject to audit, and "we believe it was trained on roughly the right
data" is not an answer that satisfies a regulator or a court. The
organization has to be able to show, concretely, which source data,
which transformations, and which version of the pipeline produced the
feature values behind a specific decision. Lineage is what makes that
showable rather than reconstructed from memory under pressure, and
building it in from the start is far cheaper than trying to retrofit it
onto years of undocumented pipeline history after an auditor asks for it.

## Why this closes out data foundations

Versioning gives you a fixed point to return to. Pipelines give you a
repeatable way to produce features. Feature stores give you one shared
implementation instead of two that can drift. Lineage ties all three
together into something you can actually interrogate: not just "we can
reproduce this" but "we can show you exactly how we got here." Together
these four practices are what let the rest of this roadmap — experiment
tracking, CI/CD, monitoring — build on data it can actually trust, rather
than data it merely assumes hasn't quietly changed underneath it.
