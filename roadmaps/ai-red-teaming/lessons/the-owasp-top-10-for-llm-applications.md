---
title: The OWASP Top 10 for LLM applications
description: A tour of OWASP's ten named LLM vulnerability categories and why a shared, versioned checklist makes red-team scoping possible.
roadmap: ai-red-teaming
node: the-owasp-top-10-for-llm-applications
group: foundations
video:
  provider: youtube
  id: "cYuesqIKf9A"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 540
level: intermediate
prerequisites: [threat-modeling-for-ai-systems]
takeaways:
  - "The OWASP Top 10 for LLM Applications names ten recurring vulnerability categories, from prompt injection to model theft, that give red teamers and developers a shared vocabulary."
  - A versioned, named list turns an open-ended "find problems" engagement into a scoping checklist both the tester and the client can agree on in advance.
  - Several of the ten categories are really about the system built around the model — plugin design, output handling, supply chain — not about the model's own weights.
  - This roadmap's later groups each correspond to one or more OWASP categories, going into far more depth than the list itself provides.
check_questions:
  - q: Why is a named, versioned top-10 list useful for an AI red-team engagement, beyond just being a reference?
    a: It gives the tester and the client a shared vocabulary and a scoping checklist, so both sides can agree in advance on which categories of issue the engagement is actually supposed to cover.
  - q: Are all ten OWASP LLM categories about flaws in the model's own weights or training?
    a: No — several, like insecure plugin design, insecure output handling, and supply chain vulnerabilities, are about the application and infrastructure built around the model rather than the model itself.
  - q: What should a reader do with each of the ten categories after finishing this lesson?
    a: Treat them as a map rather than the full territory — later groups in this roadmap take several of these categories, especially prompt injection and excessive agency, and cover them in much greater depth.
deeper:
  - label: "OWASP Top 10 for LLM Applications 2025 Full overview — Giskard"
    url: https://www.youtube.com/watch?v=j04st4edDKI
related_formulas: []
---

## Why a list, and why from OWASP

OWASP built its reputation on the Top 10 for web applications, a list so widely adopted that "OWASP Top 10" became shorthand for "the vulnerability categories any serious security review has to at least consider." When large language models started showing up in production applications with their own distinct failure modes, OWASP ran the same exercise for them: gather practitioners, aggregate what they were actually finding in the field, and name the ten categories that kept recurring. The value isn't that the list is exhaustive — it isn't — it's that it gives everyone involved in an engagement, tester and client alike, the same ten words to point at. Scoping a red-team engagement around "we want coverage of categories 1, 2, and 8" is a far more useful conversation than "try to find problems."

## The ten categories, briefly

- **Prompt injection** — getting a model to follow instructions an attacker supplied rather than the ones its operator intended, whether typed directly or smuggled in through content the model reads.
- **Insecure output handling** — trusting a model's output enough to pass it downstream unchecked, into a database query, a rendered web page, or a shell command, letting the model's text carry an injection attack into a system that trusts it.
- **Training data poisoning** — corrupting the data a model learns from so its later behavior skews toward something the attacker wants.
- **Model denial of service** — ways to degrade availability or run up cost, from resource-exhausting inputs to abusing a model's own context window.
- **Supply chain vulnerabilities** — the risk inherited from third-party models, datasets, and plugins an application builds on, the same way a vulnerable open-source library endangers any app that depends on it.
- **Sensitive information disclosure** — a model revealing private or confidential data it was trained on, given in context, or exposed to through a connected tool.
- **Insecure plugin design** — a tool or plugin that accepts the model's instructions without validating them the way any other untrusted caller would be validated.
- **Excessive agency** — granting a model more autonomy, permissions, or reach than the task in front of it actually requires.
- **Overreliance** — humans trusting a model's output without the verification its actual accuracy warrants.
- **Model theft** — an attacker copying a proprietary model's weights or behavior without authorization, whether by exfiltrating the file itself or reconstructing it through repeated queries.

## What this list is for, and what it isn't

Notice how differently shaped these ten categories are. Some describe a property of the model's own training, like poisoning. Others describe nothing about the model at all and everything about the application wrapped around it, like insecure plugin design and insecure output handling — a perfectly aligned model can still sit behind a system that trusts its output unsafely. That mix is intentional: an LLM application's real risk surface spans the model, the data, and the surrounding software, and a list that only covered one of those would miss most of what actually goes wrong in practice. Treating this as a checklist rather than a textbook is the right instinct — it tells you what to go looking for, not how deep each rabbit hole goes.

## Where the rest of this roadmap goes from here

This lesson is deliberately the shallow, naming pass. The `prompt-attacks` group that follows goes deep on prompt injection and its close cousins. `model-attacks` goes deep on poisoning, extraction, and model theft. `system-attacks` goes deep on insecure plugin design, excessive agency, and the infrastructure risk underneath supply chain vulnerabilities. Keep this list nearby as you move through the rest of the material — it's the map you'll keep returning to as a way of locating where any given attack technique actually sits.
