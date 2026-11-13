---
title: Privacy and data handling
description: How user data moves through prompts, logs and retrieval, what provider terms actually say, and designing a system that does not leak by default.
roadmap: ai-engineer
node: privacy-and-data-handling
group: safety
video:
  provider: youtube
  id: "kyJ1vd7yEPc"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 689
level: intermediate
prerequisites: [guardrails-and-moderation]
takeaways:
  - An LLM feature creates new copies of user data in prompts, traces, caches and third-party logs, and each copy is its own retention and disclosure question.
  - Stripping obvious identifiers is not anonymization — a few ordinary attributes together are often enough to re-identify someone.
  - Retrieval is an access-control surface, since an index that ignores permissions will cheerfully answer from a document the asker isn't allowed to read.
check_questions:
  - q: Why is a trace store a privacy liability as much as an engineering asset?
    a: Full traces contain the prompts and responses verbatim, so every piece of user data the feature touched now lives in a second system — usually with weaker access control and no retention policy.
  - q: Why must a RAG index enforce permissions rather than relying on the prompt?
    a: Retrieval happens before generation. If a restricted document can be retrieved, its content is already in the context, and asking the model not to use it is a request, not a control.
deeper:
  - label: "Probably Private — Beyond PII, evaluating privacy in LLM workflows"
    url: https://www.youtube.com/watch?v=WRFVtpIGBfs
related_formulas: []
---

## Count the copies

A single request makes more copies of user data than people expect. The text
goes into a prompt. The prompt travels to a provider, who may log it
depending on policy. A trace store keeps the entire exchange, verbatim, for
debugging. A cache may hold the stable prefix. An eval set may later get
built straight from these same production traces. A debugging session puts
the raw content on an engineer's screen while chasing an unrelated issue.

Each of these copies has its own lifetime, its own access control, and its
own answer — or lack of one — to "what happens to this when the user asks us
to delete their data." Most real privacy problems in LLM systems aren't
dramatic breaches. They're just a copy nobody accounted for when the system
was first designed.

## Data minimization, as concrete techniques

The strongest control is simply not transmitting sensitive data in the first
place. **Field-level redaction** strips specific fields the current task
doesn't need before anything is sent to a model — a support summarization
task rarely needs a customer's full address, even if the source ticket
contains one. **Tokenization or pseudonymization** replaces a real identifier
with an opaque reference the model can reason about structurally — "customer
4471 opened a ticket" — while the mapping back to the real identity stays on
your side and never reaches the model. Where feasible, **on-device or local
processing** for the most sensitive categories avoids transmitting them to
any external API at all, at the cost of generally needing a smaller, locally
hosted model instead of a frontier one.

## Redaction is reduction, not anonymization

Removing obvious identifiers like names and email addresses genuinely helps,
and should be standard practice. But it isn't the same as making data
anonymous, and treating it as equivalent is a common, consequential mistake.
A postcode, a date of birth, and a job title — none individually
sensitive-looking — together identify a striking majority of people in many
populations when combined. That's a well-known finding from privacy research
on exactly this kind of quasi-identifier combination.

Genuine anonymization, where it's actually required, needs considerably more
care than stripping a handful of named fields. Differential privacy — a
formal framework for adding calibrated noise so no individual record can be
confidently inferred from an aggregate result — is the serious research
answer here, though it remains uncommon in everyday LLM application work
today.

## Mapping data-subject rights to system components

Where privacy regulation grants people rights over their own data — commonly
the right to access what's held, the right to have it deleted, and the right
to a portable copy — each right has to be mapped concretely to every system
component from the "count the copies" section above, not just the primary
database. A deletion request that removes a user's row from the main
application database but leaves their data intact in a trace store, a cache,
or an eval set built from old production traces hasn't actually fulfilled the
deletion in any meaningful sense, even though the most visible copy was
handled correctly.

## Provider terms, read rather than assumed

Read the actual terms governing your specific deployment, rather than
assuming based on a provider's reputation or a blog post — and do it before
launch, not under pressure later. The load-bearing questions: is this data
used to train or improve the provider's own models; how long is it retained,
and who can access it during that window; in which jurisdiction is it
physically processed and stored; and does an enterprise agreement, where one
exists, change any of these answers relative to the default consumer or API
terms.

Consumer-facing and API-level terms from the same company are frequently
different for the same underlying model, so the relevant answer is the one
governing your actual deployment, not whatever a general search turns up. A
number of providers now offer an explicit **zero-data-retention** option for
enterprise customers, under which request content isn't persisted beyond
what's strictly needed to serve that one response — worth asking about
directly wherever the sensitivity of the data justifies it.

## Enforcing permissions where retrieval happens

In a RAG system specifically, permissions have to be enforced during the
search itself — filtering by the requesting user's actual access rights
before candidates are even returned, not afterward by hoping the generation
step declines to use something it shouldn't see. An index that can return a
document the current user isn't authorized to see has already leaked that
document's content into the context the moment retrieval completes,
regardless of what the prompt instructs the model to do with it afterward.

## What this lesson deliberately does not cover

Regional data-residency requirements and the broader contractual terms that
belong in a data-processing agreement with a vendor are genuinely important,
highly specific to the jurisdictions and industries involved, and change
often enough that anything written here in general terms would risk
misleading a reader relying on it. Where these apply to your situation,
they're properly a question for whoever at your organization handles legal
and compliance — worth raising early in a project, not discovered as a
blocker right before launch.
