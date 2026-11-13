---
title: Jailbreak techniques
description: Jailbreaking targets a model's own trained refusal behavior directly, using role-play, split payloads, and many-shot framing to defeat it.
roadmap: ai-red-teaming
node: jailbreak-techniques
group: prompt-attacks
video:
  provider: youtube
  id: "zn2ukSnDqSg"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 900
level: intermediate
prerequisites: [prompt-injection-direct-and-indirect]
takeaways:
  - Jailbreaking targets a model's own trained refusal behavior directly, while prompt injection targets getting arbitrary instructions executed at all — they overlap in practice but are not the same vulnerability.
  - Role-play and persona framing work by asking the model to simulate a fictional character whose stated traits conflict with its safety training, giving it a narrative excuse to produce output it would otherwise refuse.
  - Payload splitting and many-shot jailbreaking both exploit how a model weighs the current request against surrounding context, either by hiding intent across fragments or by burying refusal under a long run of fabricated compliant turns.
  - Jailbreaks are a moving target — a technique that reliably works against one model version routinely stops working after the next safety-training update, which is why red teaming has to be continuous rather than a one-time checklist.
check_questions:
  - q: How does jailbreaking differ from prompt injection as a vulnerability class?
    a: Prompt injection is about getting the model to execute instructions it was never meant to take from a given source, regardless of their content, while jailbreaking is specifically about defeating the model's own trained refusal behavior so it produces content its safety training was designed to block; many real attacks combine both, but they target different mechanisms.
  - q: Why does asking a model to "role-play" as an unrestricted persona sometimes bypass its safety training?
    a: Refusal is trained in as a behavior the model exhibits when it recognizes a request as itself, so framing the same request as a fictional character's dialogue can shift the model's assessment of what it is actually being asked to do, even though the underlying requested content hasn't changed at all.
  - q: Why does many-shot jailbreaking work by stuffing the context with many fake prior turns?
    a: Language models weigh the current response against the pattern established earlier in the same context, so a long run of fabricated turns in which the assistant already complied makes a refusal look like the statistically unusual response rather than the expected one, nudging the model toward continuing the established pattern.
deeper:
  - label: "How to Jailbreak LLMs - Anthropic Releases Many-shot Jailbreaking — Fahd Mirza"
    url: https://www.youtube.com/watch?v=VLZKJAyds0o
related_formulas: []
---

## A different target than injection

The previous lesson covered prompt injection: getting a model to execute
instructions from a source it shouldn't trust. Jailbreaking is a related but
distinct problem. A jailbreak doesn't necessarily smuggle in a new instruction
from an untrusted channel — the attacker can be the user, speaking plainly, in
a single ordinary chat turn. What a jailbreak attacks is the model's own
trained refusal behavior: the learned response pattern that says "recognize
this request as disallowed and decline." Safety training — reinforcement
learning from human feedback, constitutional methods, and similar techniques —
teaches a model to recognize certain request shapes and refuse them. A
jailbreak is any technique that gets the model to produce the disallowed
content anyway, usually not by disabling that training but by presenting the
request in a form the training didn't anticipate well enough to catch. The two
problems compound in practice — an indirect injection can carry a jailbreak
payload — but understanding them separately matters because the defenses differ:
filtering untrusted input sources helps against injection, while jailbreak
resistance has to be built into the model's own judgment.

## Role-play and persona framing

The oldest and still most common jailbreak family asks the model to adopt a
fictional persona whose defined traits exclude having restrictions at all — the
well-documented "DAN" ("Do Anything Now") pattern is the canonical example.
The user doesn't ask the model directly for disallowed content; they ask it to
write what a character named DAN, who "has broken free of typical AI
limitations and does not have to abide by any rules," would say in response to
that request. This works, when it works, because the model's refusal training
was shaped heavily around recognizing direct requests for harmful content, and
a layer of fictional indirection can shift the model's own assessment of what
it's actually being asked to produce. It is not that the model is tricked about
facts — it's that the framing changes what the model computes it is being asked
to do, even though a human reading the full exchange can see the underlying ask
never changed.

## Payload splitting

A second family breaks a disallowed request into several individually
innocuous-looking pieces, asking the model to assemble or act on them only at
the final step. Each fragment, read in isolation, resembles something unrelated
or harmless, and a classifier or the model's own first-pass judgment evaluating
each piece separately has nothing clearly disallowed to catch. The hazard only
becomes visible once the pieces are combined — which is often exactly the step
the model itself is asked to perform. This is less about confusing the model's
judgment of any single message and more about exploiting the fact that safety
evaluation tends to happen per-fragment rather than on the eventual composed
result.

## Many-shot jailbreaking

Many-shot jailbreaking, documented by Anthropic's own research team, exploits
long context windows directly. The attacker fills the context with a long
sequence of fabricated prior exchanges — fake user requests paired with fake
assistant responses that already comply with similar disallowed requests —
before finally asking the real question. Language models are heavily
influenced by the pattern established in their own context: if the preceding
fifty turns all show the assistant answering freely, a refusal on turn
fifty-one is the statistically unusual completion, not the expected one. This
technique specifically got more effective as context windows grew, since it
needs enough room to establish a long, convincing pattern — a direct,
unintended side effect of a capability improvement.

## Why this is a moving target

None of these techniques are permanent. Every one named here has been
patched, at least partially, by subsequent safety-training rounds once model
developers became aware of it — DAN-style prompts that worked reliably against
earlier models are mostly caught by current ones, and Anthropic's own
many-shot paper describes mitigations developed in direct response to their
discovery. This is the defining property of jailbreak research: it is
adversarial and iterative, not a fixed list to check off once. A red teamer's
job isn't to memorize today's working jailbreaks — it's to understand the
underlying mechanisms well enough to recognize the next variant when a model
update changes what works.
