---
title: Guardrails and output filtering
description: Guardrails wrap a model in classifiers and rules that screen input and output — a defense layer, not a fix to the model itself.
roadmap: ai-red-teaming
node: guardrails-and-output-filtering
group: defense
video:
  provider: youtube
  id: "IvjLXGR7-vM"
  start: 0
  end: null
creator:
  name: AI at Meta
  channel_url: https://www.youtube.com/@AIatMeta
duration_seconds: 1620
level: intermediate
prerequisites: [jailbreak-techniques, safety-filter-and-guardrail-bypasses]
takeaways:
  - Llama Guard and Prompt Guard are small, dedicated classifier models that screen inputs and outputs for unsafe content before a request reaches, or a response leaves, the main model.
  - NeMo Guardrails configures rails as rules around a model — topical rails, jailbreak rails, and fact-checking rails that can block, rewrite, or redirect a response, rather than training the behavior into the model itself.
  - A guardrail is bolted around a model rather than built into its alignment, which is exactly why the guardrail-bypass attacks covered earlier in this roadmap target the guardrail as its own separate surface.
  - Guardrails raise the cost of an attack and catch known patterns, but they do not make bypass impossible — robustness also has to be built into the model itself, which the next lesson covers.
check_questions:
  - q: How is a guardrail like Llama Guard different from safety behavior trained into the base model itself?
    a: A guardrail is a separate system — often a second, smaller classifier model — that screens input and output around the main model, rather than a property of the main model's own training.
  - q: What's the practical difference between a classifier-based guardrail and a rails engine like NeMo Guardrails?
    a: A classifier like Llama Guard gives a single unsafe-or-safe judgment on a piece of text, while a rails engine encodes configurable rules — topical, jailbreak, and fact-checking rails — that can block, rewrite, or redirect a response at multiple points in a conversation.
  - q: Why doesn't adding a guardrail fully solve the guardrail-bypass problem covered earlier in this roadmap?
    a: Guardrails catch known attack patterns and raise the cost of an attack, but techniques like Skeleton Key and Best-of-N treat the guardrail itself as a black box to search against, so a sufficiently motivated and automated attacker can often still find a bypass.
deeper:
  - label: "NVIDIA NeMo Guardrails: Full Walkthrough for Chatbots / AI — James Briggs"
    url: https://www.youtube.com/watch?v=SwqusllMCnE
related_formulas: []
---

## A layer around the model, not a property of it

Every attack covered in the `prompt-attacks` group assumed a single target:
the base model's own trained behavior. Jailbreaking works by getting the
model's own weights to override its own refusal training. But most
production LLM deployments don't ship the base model naked. They wrap it in a
second system — a guardrail — whose entire job is to inspect what goes in and
what comes out, independently of whatever the base model itself would have
done. That distinction matters because it is exactly the one the earlier
guardrail-bypass lesson depends on: defeating a guardrail is not the same
exercise as jailbreaking a model, even though a real attack chain often needs
to do both — slip past the guardrail, then get the underlying model to
comply.

Meta's own Llama Guard and Prompt Guard are a clean illustration of the
architecture. Llama Guard 3 is not a general-purpose chat model wearing a
safety hat; it's a dedicated classifier, fine-tuned specifically to read a
piece of text — a user's prompt or a model's draft response — against a fixed
taxonomy of unsafe content categories and return a verdict. Prompt Guard does
a narrower, faster version of the same job aimed at the input side: is this
text a prompt-injection or jailbreak attempt, as opposed to a normal request?
Both run as separate inference calls, before or after the main model's own
generation, and both can be deployed in front of any underlying chat model,
not just Meta's own. That's the structural point worth holding onto: a
guardrail is infrastructure added around a model, portable across whichever
model happens to be behind it, not a capability that lives inside one
specific model's weights.

## Classifiers versus rails engines

Llama Guard-style classifiers answer one narrow question well: is this text,
in isolation, unsafe? NeMo Guardrails, NVIDIA's open-source framework, takes
a different shape — it's a rules and configuration layer, not a single model
call. A deployment using NeMo Guardrails defines "rails": topical rails that
keep a customer-support bot from answering questions outside its product
scope, jailbreak rails that pattern-match or classify an incoming message
against known jailbreak shapes before it ever reaches the main model, and
fact-checking or hallucination rails that can compare a generated answer
against a source document before it's shown to the user. Crucially, a rail
can do more than classify and block — it can rewrite a response, substitute a
canned safe answer, or redirect the conversation into a different flow
entirely. A classifier gives you a verdict; a rails engine gives you a
pipeline of configurable interventions wired around the model's input and
output at multiple points in a conversation, not just a single pass-or-fail
gate.

## Defense in depth, not a solved problem

The guardrail-bypass lesson earlier in this roadmap covered Skeleton Key —
wording a request so disallowed content gets prefaced with a warning label
instead of refused, satisfying a filter's literal rule while defeating its
purpose — and Best-of-N, which treats a guardrail as a black box and
brute-forces variants of a prompt until one slips through. Both techniques
exist precisely because a guardrail is a separate, inspectable, and
ultimately gameable system sitting in front of the model, with its own
failure modes independent of the base model's. A classifier trained on known
jailbreak phrasing will systematically miss phrasing it wasn't trained on; a
rails engine's topical rail only blocks what its configuration anticipated.
None of this makes guardrails worthless — they catch the overwhelming
majority of casual and known-pattern abuse cheaply, and they raise the bar
for a would-be attacker from "type anything" to "find or construct a
bypass." But "raises the cost of an attack" and "prevents the attack" are
different claims, and conflating them is how teams end up overconfident in a
single layer of defense. That's why the next lesson moves from guardrails
bolted around a model to robustness trained into the model itself — a deeper,
harder-to-retrofit layer that doesn't rely on an external system correctly
recognizing an attack it may never have seen before.
