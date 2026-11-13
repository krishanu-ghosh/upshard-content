---
title: Responsible disclosure and reporting
description: A finding that's never reported, or reported badly, has the same effect as no testing — this lesson covers doing it right.
roadmap: ai-red-teaming
node: responsible-disclosure-and-reporting
group: defense
video:
  provider: youtube
  id: "Pz_J1OuXp70"
  start: 0
  end: null
creator:
  name: FIRST
  channel_url: https://www.youtube.com/@FIRSTdotorg
duration_seconds: 1740
level: intermediate
prerequisites: [continuous-monitoring-and-detection, building-a-red-team-lab]
takeaways:
  - Responsible disclosure for an AI vulnerability means reporting privately to the vendor or model provider first and giving them a fair remediation window before any public writeup, the same norm that governs conventional software vulnerability disclosure.
  - FIRST, the organization behind CVSS, is extending CVE-style identity and severity scoring to AI and ML models, so a found vulnerability gets a real identifier and a comparable severity rating instead of an informal heads-up to a vendor.
  - A useful red-team report gives clear reproduction steps and concrete, demonstrated impact, not just a claim that the model said something bad once under unspecified conditions.
  - Model cards document a model's known risks, limitations, and intended use cases up front, so a vulnerability doesn't have to be independently rediscovered by every downstream user before anyone acts on it.
check_questions:
  - q: What does responsible disclosure require of a red teamer who finds a serious AI vulnerability, before publishing anything about it?
    a: They should report the finding privately to the vendor or model provider first and give them a reasonable window to investigate and remediate before any public disclosure, coordinating the timing rather than publishing immediately.
  - q: Why is FIRST's work extending CVE-style identifiers to AI vulnerabilities useful, beyond just giving a finding a name?
    a: It gives an AI vulnerability a consistent identity and a comparable severity score the way CVSS does for conventional software vulnerabilities, which lets vendors, researchers, and downstream users track, prioritize, and reference the same finding instead of relying on informal, inconsistent descriptions.
  - q: What separates a red-team report that's actually useful from one that's just alarming?
    a: A useful report includes clear, specific reproduction steps and a concrete description of real impact, while an alarming-but-useless report just claims the model produced something bad once without enough detail for anyone to verify, reproduce, or fix it.
  - q: What problem do model cards solve that disclosure alone doesn't?
    a: Model cards proactively document a model's known risks and limitations before anyone has to find them the hard way, so users and downstream developers can make informed decisions without needing a vulnerability to be independently rediscovered first.
deeper:
  - label: "What a Good AI Red Team Report Actually Looks Like — Zero Day Security | AI & Cyber Risk"
    url: https://www.youtube.com/watch?v=8ieWhjw_RxA
  - label: "Model Cards for Model Reporting — ZazenCodes"
    url: https://www.youtube.com/watch?v=saAUB_MG2d0
related_formulas: []
---

## Closing the loop: why a finding only matters once it's reported

Twenty-some lessons of this roadmap have been about finding things that are
wrong with an AI system — how to think about its attack surface, how to probe
it, how to automate that probing, how to defend against what gets found. None
of that work does anything for anyone if the finding stops at "I got the
model to do something bad once" and goes no further. An unreported
vulnerability, or one reported so vaguely nobody can act on it, has exactly
the same practical effect on the system's actual safety as a vulnerability
that was never looked for at all. This last lesson is about the unglamorous,
often-skipped final step of an engagement: turning a finding into something a
vendor can actually fix, on a timeline, through a process attackers outside
your engagement can't exploit in the meantime.

## Giving a vulnerability an identity: FIRST and CVE-style scoring for AI

For decades, conventional software vulnerabilities have had shared infrastructure around them: a CVE identifier that names the specific flaw uniquely, and a CVSS score, maintained by FIRST, giving it a comparable severity rating. A vendor, a security team, and a downstream user can all reference the same finding and reason about how urgent it is. AI vulnerabilities have historically lacked this. A jailbreak or data-leakage finding against a model would get reported as an informal email or a blog post — no consistent identifier, no standard severity scale, no reliable way to tell whether two reports from different researchers described the same underlying flaw. FIRST's work extending CVE-style identification to AI and ML models treats a found AI vulnerability the way any other serious software vulnerability has long been treated: it gets a real identity, a severity assessment against criteria the field agrees on, and a place in a shared record other people can check before re-discovering the same thing independently. That's not bureaucracy for its own sake. It's what lets an ecosystem of vendors, researchers, and downstream deployers of the same model coordinate a response at all.

## What makes a report useful instead of just alarming

Having an identifier and a score only matters if the underlying report is
actually usable, and a surprising number of red-team write-ups fail at
exactly this step. "I got the model to produce something bad" is a headline,
not a finding — it tells a vendor nothing about how to reproduce it, under
what conditions it's reliable, or what realistic harm follows from it. A
genuinely useful report gives precise reproduction steps — the exact prompt
or sequence of turns, the model version, any system-prompt context that
mattered — states plainly how reliable the exploit is across repeated
attempts rather than reporting a single lucky hit, and describes concrete
impact in terms the vendor's engineering and policy teams can actually weigh
against a fix's cost. The difference between a report that gets prioritized
and one that gets ignored is almost always this level of specificity, not the
raw severity of the underlying issue.

## Model cards: disclosure before anyone has to ask

Everything above describes disclosure after a vulnerability is found. Model
cards are the complementary, proactive half of the same idea: documentation,
published alongside a model, that states its known risks, limitations, and
intended use cases before any external party has to go digging for them. A
model card that honestly says "this model has not been evaluated for use in
medical decision-making" or "this model is known to be susceptible to
prompt injection via tool outputs" does real work — it shifts the burden of
discovering a limitation away from every individual downstream user
independently rediscovering it the hard way, and it gives a red teamer
testing the system a documented baseline to test against rather than a blank
slate. Responsible disclosure and model cards are two sides of the same
discipline: being honest, specifically and usably, about what's actually
wrong or might go wrong with a system — not because disclosure is pleasant,
but because a finding that never gets communicated clearly to the people who
can fix it accomplishes nothing at all.
