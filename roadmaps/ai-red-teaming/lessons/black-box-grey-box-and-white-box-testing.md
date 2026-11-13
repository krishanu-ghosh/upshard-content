---
title: Black-box, grey-box, and white-box testing
description: What a red teamer can see — public interface only, partial internals, or full source — determines what an AI red-team engagement can actually find.
roadmap: ai-red-teaming
node: black-box-grey-box-and-white-box-testing
group: methodology
video:
  provider: youtube
  id: "e4GQNN5EG4E"
  start: 0
  end: null
creator:
  name: OWASP Foundation
  channel_url: https://www.youtube.com/@OWASPGLOBAL
duration_seconds: 2100
level: intermediate
prerequisites: [rules-of-engagement-and-ethics]
takeaways:
  - "Access level is a scoping decision made before testing starts, and it determines which classes of vulnerability an engagement can realistically surface."
  - Black-box testing simulates exactly what an outside attacker sees — the chat window or public API and nothing else — so it is the most realistic test of real-world exposure.
  - "White-box testing trades realism for depth: full access to weights, prompts, and training data finds structural flaws a black-box attacker might never stumble onto."
  - Grey-box testing, with partial access such as a known system prompt or tool list, often models the most common real-world threat more accurately than either extreme.
check_questions:
  - q: Why would a red team deliberately choose black-box access instead of just taking the full white-box access that would find more bugs?
    a: Because black-box testing measures what a real external attacker can actually exploit with no insider knowledge, which is the risk that matters most for most deployed systems, and a finding that only white-box access could surface may not be reachable by anyone outside the organization.
  - q: What makes grey-box testing a realistic simulation of an insider or leaked-system-prompt scenario?
    a: A grey-box tester starts with exactly the kind of partial knowledge — a known system prompt, a known tool list, maybe API documentation — that a disgruntled employee, a contractor, or an attacker who found a leaked prompt online would actually have, rather than either total ignorance or full source access.
  - q: How does access level change what a red-team report can claim?
    a: A black-box finding can be reported as "an unauthenticated external attacker can do this today," while a white-box finding has to be reported as a structural weakness that is real but may require insider access or further work to reach from outside.
deeper: []
related_formulas: []
---

## Access level is a scoping decision, not an afterthought

Before anyone sends a single adversarial prompt, an AI red-team engagement has to answer a basic question: how much does the tester get to see? This is borrowed vocabulary from conventional penetration testing, but it maps onto AI systems with a few important twists. The three points on the spectrum are black-box, white-box, and grey-box. The choice between them isn't a technical detail — it's a scoping decision that determines what kinds of weaknesses the engagement is even capable of finding, and how the resulting report should be read.

## Black-box: testing what an attacker actually has

In a black-box engagement, the tester gets exactly what a member of the public gets: the chat interface, or the documented API, and nothing else. No system prompt, no model weights, no architecture diagram, no list of which tools the agent can call. Every finding has to be earned the hard way — probing the running system and inferring its behavior from its outputs, the same position a real opportunistic attacker is in.

This is why black-box results carry a particular kind of credibility. If a black-box tester gets the system to leak its system prompt or bypass a content filter, that's proof an attacker with zero inside knowledge could do the same thing today. The tradeoff is coverage: a black-box tester can spend weeks probing and never stumble onto a flaw that's obvious the moment you see the actual system prompt or the retrieval pipeline's configuration.

## White-box: full access, structural depth

A white-box engagement is the opposite extreme. The tester has the model weights, the system prompt, the training or RAG data, the orchestration code, sometimes even access to run the model locally without rate limits or logging. This access buys depth that black-box testing cannot reach. A tester who can read the system prompt directly can find every exploitable ambiguity in its wording instead of guessing at it through trial and error. A tester with the weights can study the model's behavior offline, develop an adversarial suffix or a jailbreak with unlimited attempts, and only fire the polished version at the live system.

White-box testing is how you find deep structural issues — a RAG pipeline that trusts retrieved documents with no provenance check, a tool definition with excessive permissions baked into its schema — that an external attacker might never discover, but that a sufficiently motivated or lucky one eventually could. The honest caveat belongs in the report: a white-box finding proves the flaw exists, not that it's trivially reachable from outside.

## Grey-box: the realistic middle

Grey-box testing gives the tester some internal knowledge but not all of it — commonly the system prompt and the list of tools an agent can call, but not the model weights or the full source. That sounds like a compromise, but it's frequently the most realistic simulation available, because it matches how real incidents actually happen. System prompts get leaked onto forums and pasted into screenshots routinely. A contractor or former employee knows the tool list without ever having seen the weights. A competitor can often reverse-engineer a tool's behavior from its outputs without ever touching source code.

Grey-box testing lets a red team ask a specific question: given the partial knowledge a realistic adversary is likely to already have, what can they do with it?

## Matching access level to what the report needs to say

None of the three is universally "better" — they answer different questions. A mature red-team program typically runs more than one across the lifetime of a system: an initial white-box pass to find and fix structural issues before launch, recurring black-box testing against the live, deployed system to confirm those fixes hold up from a real external vantage point, and grey-box testing filling the gap whenever the realistic threat model includes an insider or a leaked prompt.

Reporting has to track this precisely. Collapsing a white-box structural finding into language that implies any outsider could reproduce it overstates the risk. Describing a black-box finding cautiously when it was in fact demonstrated against the live production system with no special access understates it.
