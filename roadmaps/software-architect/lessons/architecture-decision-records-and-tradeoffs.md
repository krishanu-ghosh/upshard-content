---
title: "Architecture Decision Records and How to Argue a Trade-off"
description: What an ADR actually records, the template that makes one useful, and how to revisit a decision later without relitigating it from scratch.
roadmap: software-architect
node: architecture-decision-records-and-tradeoffs
group: the-architect-role
video:
  provider: youtube
  id: "LMBqGPLvonU"
  start: 0
  end: null
creator:
  name: Software Architecture Monday
  channel_url: https://www.youtube.com/@markrichards5014
duration_seconds: 628
level: intermediate
prerequisites:
  - levels-of-architecture
takeaways:
  - "An ADR records one decision, the context that forced it, the options actually considered, and the consequences accepted — not a general design document, and not a decision nobody disagreed with."
  - "The context section is the part most often skipped and the part that matters most later — without it, a decision reads as arbitrary to anyone reading it after the constraints that justified it have changed or been forgotten."
  - "An ADR is immutable once accepted — if circumstances change, the fix is a new ADR that supersedes the old one and says why, not an edit that erases the trail of what used to be true."
  - "Recording the options that were rejected, and why, is what actually prevents a decision from being relitigated every few months by someone who wasn't in the original conversation."
  - "Not every decision deserves an ADR — reserve them for decisions that are expensive to reverse or that future engineers will reasonably ask \"why is it built this way\" about; recording a reversible, low-stakes choice is noise that drowns out the ones worth reading."
check_questions:
  - q: What goes wrong when an ADR's context section is skipped or written too thin?
    a: Without the constraints that forced the decision, anyone reading it later — once the environment has changed — can't tell whether the decision is still correct or was already outdated and simply never revisited, so they either trust a stale choice or re-derive the reasoning from scratch.
  - q: Why is an accepted ADR treated as immutable rather than edited when the decision changes?
    a: Editing it erases the historical record of what the team actually believed and why at the time; the correct fix is a new ADR that explicitly supersedes the old one, so both the original reasoning and the reason it changed stay visible.
  - q: Why record the alternatives an ADR rejected, not just the option chosen?
    a: It stops the decision from being relitigated every time someone new joins who would have suggested the same rejected option — the ADR already shows it was considered and why it lost, so the conversation can move straight to whether circumstances have actually changed.
deeper:
  - label: "Using Architecture Decision Records – Jonathan Wolski"
    url: "https://www.youtube.com/watch?v=MQJUWtTM1-E"
related_formulas: []
---

## The problem an ADR solves

Every system accumulates decisions that aren't visible in the code: why Postgres and not DynamoDB, why this service calls that one synchronously instead of through a queue, why a particular library was ruled out. Six months later, none of that reasoning survives in anyone's memory, and a new engineer — or the same engineer, having forgotten — proposes reopening a decision that was already carefully made, for reasons that no longer exist anywhere to be checked against. An **architecture decision record (ADR)** is a short, durable document that exists specifically to survive that gap: one file per significant decision, committed alongside the code it governs, answering "why is it built this way" without anyone having to ask.

## What actually goes in one

Michael Nygard's original template, still the most common one in practice, has four parts, and skipping any of them is where most ADRs go wrong:

- **Title and status.** A short, specific title ("Use Postgres as the primary datastore for the orders service") and a status — proposed, accepted, deprecated, or superseded by a later ADR.
- **Context.** The forces that made this a decision at all: the constraint, the requirement, the competing pressures. This is the section people skip because it feels like preamble, and it's the one that matters most a year later — without it, a decision reads as arbitrary the moment the environment around it has shifted, because there's nothing left to check whether the original forces still apply.
- **Decision.** The choice itself, stated plainly, in active voice — "we will use Postgres" — not hedged or described as a leaning.
- **Consequences.** What this decision costs as well as what it buys. Every real architectural decision trades something away; an ADR that only lists benefits is marketing, not a record. If choosing Postgres means giving up a schema-less document model some future feature might have wanted, that trade-off belongs here, explicitly, so nobody later feels blindsided by a cost that was actually known and accepted up front.

A fifth section worth adding even though Nygard's original template doesn't require it: **alternatives considered**, with one line each on why they lost. This is the single most valuable addition, because it's what actually stops the decision from being relitigated — the next person who would have proposed the same rejected option can see it was already weighed, and the conversation moves straight to "has anything actually changed" instead of starting over.

## Arguing the trade-off itself

Writing the Decision and Consequences sections well requires actually doing the trade-off analysis, not asserting a conclusion. A trade-off worth recording usually has:

- **A forcing constraint** — a real requirement (latency budget, team size, compliance rule, existing skill set) that rules some options out, not just a preference.
- **At least two live options**, not a straw-man alternative set up to lose. If only one option was ever seriously considered, that's worth saying honestly in the context rather than inventing a fake comparison.
- **An explicit axis of comparison** — cost, complexity, time to ship, operational burden, how reversible it is — because "which is better" without naming what "better" means for this decision is not an argument, it's an opinion.
- **A decision that names what's being given up**, not just what's being gained. "We chose managed Postgres over self-hosted MySQL because our team has no DBA and can't own patching and failover ourselves" is a real trade-off statement; "Postgres is the industry standard" is not.

## Revisiting a decision without relitigating it

Requirements change, and a decision that was correct eighteen months ago can become wrong without anyone having done anything wrong. The discipline that keeps ADRs useful here is: **never edit an accepted ADR to reflect the new reality.** Write a new one, mark it as superseding the old, and state in its context exactly what changed since the original was accepted. This preserves two things simultaneously that an edit would destroy: the historical record of what the team reasonably believed at the time, given what they knew then, and a clear, dated trail of how the system's reasoning evolved — which is often exactly what a future architect needs to tell whether today's constraints are actually new or were already anticipated and dismissed once before.

## Knowing when not to write one

ADRs lose their value the moment they stop being selective. A team that writes one for every choice, including fully reversible ones nobody will ever ask about, ends up with a backlog nobody reads, which defeats the entire point — a record that exists but isn't consulted is no better than no record at all. The practical filter: write one when the decision is expensive to reverse, or when a reasonable future engineer would ask "why is this built this way" and deserve a real answer. Everything else can stay a comment in the code or a line in a pull request description.
