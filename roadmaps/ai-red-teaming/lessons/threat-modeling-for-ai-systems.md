---
title: Threat modeling for AI systems
description: Threat modeling an AI system means naming its real assets and adversaries before testing it — the prompt itself is now an entry point.
roadmap: ai-red-teaming
node: threat-modeling-for-ai-systems
group: foundations
video:
  provider: youtube
  id: "oh_Uibdmk7c"
  start: 0
  end: null
creator:
  name: KryptoMindz Technologies
  channel_url: https://www.youtube.com/@kryptomindz
duration_seconds: 900
level: intermediate
prerequisites: [what-is-ai-red-teaming]
takeaways:
  - "Threat modeling means naming, before any testing starts, what an AI system's real assets are, what its entry points are, and who would want to abuse it and why."
  - In an LLM-based system, the asset is rarely just "the model" — it can be the system prompt, the training or retrieval data, the backend actions a tool can take, or simply end-user trust in the output.
  - A prompt is an input channel an attacker can reach indirectly, through a retrieved document or a tool's return value, not only by typing directly into the chat box.
  - Mapping assets, entry points, and adversary goals before testing turns an open-ended "try to break it" exercise into a scoped, prioritized one.
check_questions:
  - q: What counts as "the asset" when threat modeling an AI system?
    a: It depends on the system, but it is rarely just the model's weights — it is often the system prompt, the training or retrieval data behind it, the actions its tools can take, or the end user's trust in what it tells them.
  - q: Why is the prompt itself considered part of the attack surface in a way a normal web form isn't?
    a: Because a model can be reached indirectly — an attacker can plant an instruction inside a document that gets retrieved, a web page a tool fetches, or any other content the model is asked to process, not only by typing directly into the chat.
  - q: What are the three things a threat model should produce before testing an AI system begins?
    a: A list of its real assets worth protecting, a map of every entry point that can reach the model or its surrounding system, and a sense of which adversaries would target it and what they would be trying to achieve.
deeper:
  - label: "Adam Shostack's Threat Modeling Intensive with Complete AI At Blackhat 2026 — Adam Shostack"
    url: https://www.youtube.com/watch?v=iNms-govBaI
related_formulas: []
---

## Threat modeling is not optional scaffolding

It's tempting to skip straight to the fun part of red teaming — trying prompts, watching what breaks. But without a threat model, that exercise has no way to know when it's done or what actually mattered. Threat modeling is the discipline of answering three questions before any probing starts: what are we actually trying to protect, how can an adversary reach it, and who's likely to try, with what goal in mind. None of this is specific to AI — it's the same process security teams have run against web applications and networks for decades. What changes for an AI system is the answer to each question, and that's worth working through carefully rather than assuming the old answers still apply.

## What counts as an asset in an AI system

In a conventional application, the asset is usually obvious: a database of customer records, a payment flow, an admin panel. In an AI system the asset is less obvious and often plural. The model's weights themselves can be an asset — expensive to train, worth stealing. The system prompt is an asset, because it often encodes business logic, guardrails, or proprietary instructions an operator would rather not expose. Training data or a RAG pipeline's document store is an asset, both as a confidentiality concern (it may contain sensitive records) and an integrity concern (poisoning it changes future answers). And less tangibly, but just as real, end-user trust in the system's output is an asset: a support bot that can be tricked into giving a wildly wrong refund policy damages trust and causes real financial harm even without "hacking" anything in the conventional sense. A good threat model names all of the assets actually in play for a given system, rather than defaulting to "the model" as a single catch-all answer.

## What's genuinely new about the attack surface

The sharpest difference from a normal web application is where input can come from. A web form has one obvious entry point: the field a user types into, which a scanner can enumerate completely. An LLM-based system has that same direct channel, but it also treats *anything the model reads* as a potential instruction channel, whether or not a human typed it. A RAG pipeline retrieves a document and feeds its full text to the model as context; if an attacker can get content into that document store — editing a wiki page, planting a comment, uploading a file — they've reached the model without ever touching its chat interface. An agent that browses the web or reads email inherits the same problem: the page it fetches or the message it's asked to triage is itself untrusted input, carrying as much potential influence as anything typed directly into the prompt. Mapping every one of these indirect paths, not just the obvious chat box, is the entry-point half of the threat model.

## A worked example: threat-modeling a RAG chatbot

Take a simple customer-support chatbot built over a RAG pipeline: it retrieves from a knowledge base of help-center articles and answers questions in that voice. The assets are the article corpus (its accuracy and provenance), the system prompt (which may include escalation and refund rules), and user trust in the answers given. The entry points are the obvious chat box, plus anything that can alter the knowledge base — if customers or employees can submit or edit articles, that's a second entry point reaching the model indirectly. The adversaries worth considering range from a curious user probing for amusement, to a competitor trying to get the bot to say something embarrassing, to an insider or compromised account editing an article to quietly change the refund policy the bot quotes. Only after laying this out does it make sense to decide what to actually test first — and that prioritization is the entire point of doing this before, not during, the engagement.
