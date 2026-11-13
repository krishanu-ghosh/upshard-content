---
title: What is AI red teaming?
description: AI red teaming probes a deployed model's behavior for ways it can be made to misbehave — a discipline distinct from pentesting and ML evaluation.
roadmap: ai-red-teaming
node: what-is-ai-red-teaming
group: foundations
video:
  provider: youtube
  id: "DabrWAKNQZc"
  start: 0
  end: null
creator:
  name: Microsoft Developer
  channel_url: https://www.youtube.com/@MicrosoftDeveloper
duration_seconds: 900
level: beginner
prerequisites: []
takeaways:
  - AI red teaming probes a deployed AI system's behavior for ways it can be made to act badly, rather than testing whether the underlying model is simply accurate.
  - It is a different discipline from traditional penetration testing, which targets code and infrastructure, because the primary attack surface here is natural language itself.
  - It is also different from ML robustness or accuracy evaluation, which measures how well a model performs its intended task under normal conditions.
  - A real engagement looks like an open-ended, creative conversation with a chatbot or agent, not a scripted exploit run against a compiled binary.
check_questions:
  - q: How does AI red teaming differ from traditional penetration testing?
    a: Traditional pentesting hunts for exploitable bugs in code and infrastructure, while AI red teaming targets a deployed model's behavior and outputs, probing it with natural language to see what it can be made to say or do.
  - q: Why isn't ML robustness evaluation the same thing as AI red teaming?
    a: Robustness evaluation measures how accurately a model performs its intended task; red teaming deliberately tries to make it misbehave or produce unsafe output, independent of how accurate it otherwise is.
  - q: What does a typical AI red-team session actually look like?
    a: It looks like an extended, improvisational conversation with a chatbot or agent — trying unusual phrasings, personas, and follow-up turns — rather than a one-shot exploit fired at a binary.
deeper:
  - label: "AI Red Teaming, explained — Group-IB"
    url: https://www.youtube.com/watch?v=yXorivjZ8Pg
related_formulas: []
---

## A new kind of target

Security testing has always meant trying to break something before someone else does. AI red teaming applies that instinct to a new kind of target: a deployed AI system, usually built on top of a large language model, that takes natural-language input and decides what to say or do in response. The thing being attacked is not a binary with a buffer overflow waiting somewhere in its memory layout. It is a system whose entire surface is conversational — a chat window, an API that accepts a prompt, an agent that reads documents and calls tools. What a red teamer is hunting for is not a crash or a privilege escalation in the classic sense, but a way to make the system say something it shouldn't, reveal something it shouldn't, or act in a way nobody authorized.

## Why this is not traditional penetration testing

A conventional penetration test targets the software and infrastructure around an application: unpatched dependencies, misconfigured servers, SQL injection in a form field, a weak authentication flow. The vulnerabilities are, in principle, fixable facts about code. AI red teaming overlaps with that world — a model-serving stack still runs on real servers with real APIs — but its defining activity sits one layer up, in the model's behavior itself. Getting a chatbot to ignore its instructions, output content it was explicitly trained to refuse, or leak a fragment of its system prompt is not a bug in the ordinary sense; there is no single line of code to patch. The "vulnerability" is a property of how the model generalizes from its training and alignment, which makes it fuzzier, more input-dependent, and much harder to close off permanently than a missing patch.

## Why this is not ML evaluation either

It is also easy to confuse AI red teaming with standard machine learning evaluation, and the two get run by genuinely different teams for a reason. Evaluation asks "how good is this model at the task it was built for?" — accuracy, calibration, benchmark scores, average-case performance. Red teaming asks a hostile question instead: "what is the worst this model can be made to do, on purpose, by someone trying?" A model can score well on every standard benchmark and still be trivially coaxed into producing something dangerous, biased, or false, because nothing about benchmark accuracy measures resistance to an adversary who is actively searching for the failure case. Red teaming is worst-case thinking applied to model behavior, not average-case measurement.

## Why this discipline exists now

Natural language becoming an attack surface is genuinely new. A web form has a finite, well-understood set of input types a scanner can enumerate. A prompt box accepts anything expressible in language, including instructions disguised as data, personas designed to unlock refused behavior, and content lifted from documents or tool results the model was never told to distrust. Earlier generations of software didn't execute instructions found inside their own inputs; language models, by design, do exactly that — the model's job is to read text and act on what it means. That single property is why AI red teaming had to become its own discipline rather than an extension of existing practice.

## What an engagement actually looks like

In practice, a red-team session looks less like a CTF against a hardened binary and more like a sustained, improvisational conversation. A tester might open with an innocuous request, watch how the system responds, then adjust phrasing, introduce a persona, reference an earlier turn, or embed an instruction inside a document the system is asked to summarize — iterating the way a conversation partner iterates, not the way an exploit script iterates. That loop of observe, adapt, and try again is the actual day-to-day shape of the work this entire roadmap builds toward.
