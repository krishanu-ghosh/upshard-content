---
title: Rules of engagement and ethics
description: Scope, written authorization, and hard limits before testing an AI system — and why AI red-team ethics go beyond traditional pentest rules.
roadmap: ai-red-teaming
node: rules-of-engagement-and-ethics
group: foundations
video:
  provider: youtube
  id: "jcJRPZtX23w"
  start: 0
  end: null
creator:
  name: DEFCONConference
  channel_url: https://www.youtube.com/@DEFCONConference
duration_seconds: 2700
level: beginner
prerequisites: [what-is-ai-red-teaming]
takeaways:
  - "A red-team engagement needs written authorization and a defined scope before any testing starts — without it, probing a system for failures is indistinguishable from attacking it."
  - Scope has to name both what's in bounds (which systems, which techniques) and what stays off-limits even with full authorization, such as testing against real third-party victims or generating content with no testing value at all.
  - Testing whether a model can be made to produce genuinely harmful content is itself an ethically loaded act, unlike testing whether a firewall rule can be bypassed, because the act of eliciting the content can itself cause harm.
  - DEF CON 31's Generative Red Team Challenge showed these principles operating at a scale no prior AI red-team exercise had attempted — thousands of public participants against real deployed models under a single authorized scope.
check_questions:
  - q: What must be in place before any AI red-team testing begins?
    a: Written authorization from whoever owns the system, along with a clearly defined scope stating which systems and techniques are in bounds and which are off-limits.
  - q: Why can something stay off-limits in an AI red-team engagement even with full authorization to test?
    a: Because authorization to test a system doesn't automatically justify every possible action against it — testing against real third-party victims, or eliciting content with no diagnostic value, can cause harm the authorization was never meant to cover.
  - q: How does AI red-team ethics differ from the ethics of a traditional penetration test?
    a: Testing a firewall rule or a login form either succeeds or fails with no ethical weight in the failure itself, but testing whether a model will produce genuinely harmful content means the tester may actually generate that harmful content in the process, which carries its own ethical cost independent of the test's result.
  - q: What made DEF CON 31's Generative Red Team Challenge notable as an example of scoped testing?
    a: It authorized thousands of members of the public, not vetted insiders, to attack real deployed models live, which meant the organizers had to define scope, authorization, and harm boundaries carefully enough to hold up at a scale no earlier AI red-team exercise had attempted.
deeper:
  - label: "Panel: GenAI Red Teaming guidance and best practices — OWASP GenAI Security Project"
    url: https://www.youtube.com/watch?v=VYDSIhskmEM
related_formulas: []
---

## Authorization comes first, every time

Everything this roadmap teaches is dual-use. The same techniques that make someone a careful AI red teamer make them a capable attacker if pointed at a system they have no right to touch. The line between the two isn't skill or intent in the abstract — it's authorization. Before a single probe is sent, a red-team engagement needs explicit, written permission from whoever actually owns the system being tested, naming the tester, the time window, and the systems in scope. Without that document, trying to make a production chatbot misbehave isn't a security exercise. It's simply attacking someone's live system. This is the same baseline rule traditional penetration testing has always operated under, and nothing about AI changes it.

## Scope has to say what's out, not just what's in

A scope document that only lists what's permitted is incomplete. The more important half, often skipped by newcomers, is stating explicitly what stays off-limits even with full sign-off. A few categories come up constantly in AI engagements specifically. Testing is scoped to the system under test, not to any real third party whose name, data, or likeness might surface during probing — a jailbreak attempt that happens to generate harassment targeting an actual person is out of bounds regardless of what the authorization covers. Techniques that risk real, uncontained harm need their own explicit carve-out or containment plan, not an assumption that "testing" excuses everything: material that could cause damage if it leaked beyond the test environment, or probing that could affect real users of a shared production system. And engagements should stop generating harmful content the moment it has proven the point — continuing past that for volume or shock value serves no diagnostic purpose and simply accumulates the harmful material itself.

## Why this ethical weight is different from traditional pentesting

A conventional penetration tester who finds that a firewall rule lets traffic through hasn't created anything harmful in the process of finding it. The finding is a fact about a misconfiguration, with no payload left behind once the report is filed. AI red teaming doesn't have that luxury. To prove a model can be coaxed into producing dangerous instructions, biased output, or content that harms a real person, the tester typically has to actually produce it. The test and the harm are the same action, separated only by what happens to the output afterward — whether it's logged, contained, and reported responsibly, or allowed to spread. That makes informed consent, careful handling of generated material, and restraint about what gets tested in the first place genuinely load-bearing, in a way they rarely are for a firewall audit. An AI red teamer isn't just looking for failures. In a controlled and authorized way, they're manufacturing the exact failure they're studying.

## Seeing these principles at scale

DEF CON 31's Generative Red Team Challenge, organized by Sven Cattell, Rumman Chowdhury, and Austin Carson, is the clearest public illustration of what happens when these principles have to hold up under real pressure. Rather than a small team of vetted professionals, the event opened real, deployed language models to thousands of members of the public over a structured scope. Making that work at all meant defining, in advance and in detail: which prompts and techniques were in scope, what harm boundaries no participant could cross regardless of what they discovered, and how findings would be collected and handled afterward rather than left to spread unmoderated. It's a useful case precisely because the organizers had to solve, at unprecedented scale, the exact scoping and harm-boundary problems every smaller engagement faces. It's proof that these rules aren't bureaucratic overhead — they're what makes testing at any scale possible without the exercise becoming the harm it was meant to study.
