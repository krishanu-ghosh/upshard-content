---
title: Multi-turn and encoding attacks
description: Crescendo escalates gradually across turns while encoding tricks disguise text — both exploit filters that judge surface text, not final intent.
roadmap: ai-red-teaming
node: multi-turn-and-encoding-attacks
group: prompt-attacks
video:
  provider: youtube
  id: "jedyIGDIJ24"
  start: 0
  end: null
creator:
  name: Giskard
  channel_url: https://www.youtube.com/@giskard_ai
duration_seconds: 600
level: advanced
prerequisites: [jailbreak-techniques, safety-filter-and-guardrail-bypasses]
takeaways:
  - Crescendo is a multi-turn jailbreak that starts with clearly acceptable requests and escalates gradually, exploiting a model's tendency to stay consistent with its own prior turns rather than re-evaluating the full conversation from scratch each time.
  - Encoding and obfuscation attacks disguise disallowed content using base64, leetspeak-style substitution, translation into a less-moderated language, or splitting a flagged term across tokens and whitespace, so pattern-matching filters never see the literal flagged text.
  - Both technique families succeed for the same underlying reason — a safety system that evaluates one message or one surface string in isolation, instead of the model's eventual action read in full conversational context, is blind to intent that only becomes visible once pieces are assembled.
  - Defending against either requires evaluating meaning and trajectory, not just matching literal text against a blocklist in a single message.
check_questions:
  - q: Why does Crescendo succeed even when a direct, single-turn version of the same final request would be refused?
    a: Crescendo builds the request gradually across many turns, each one only a small escalation from an already-accepted prior turn, which exploits the model's tendency to weigh conversational consistency with its own recent responses rather than re-evaluating the full trajectory and intent of the conversation against its safety training at every single step.
  - q: Why does encoding a disallowed phrase in base64, or splitting it with extra whitespace, evade a simple keyword filter?
    a: Encoding a disallowed word or phrase in base64, or splitting it across extra whitespace and punctuation, means the literal flagged string never appears in the surface text a keyword or pattern-matching filter scans, even though the model itself can still decode or reconstruct the original meaning and act on it.
  - q: What single underlying weakness do Crescendo and encoding attacks both exploit?
    a: Both exploit safety systems that evaluate a single message's surface text in isolation rather than the model's eventual action read in full context, so intent that is only visible across multiple turns, or hidden behind a transformation on the text itself, slips past evaluation that only looks at one message's literal wording at a time.
deeper:
  - label: "Episode 5: Prompt Injection Attacks – Single-Turn | AI Red Teaming 101 — Microsoft Developer"
    url: https://www.youtube.com/watch?v=jle327dpdpw
related_formulas: []
---

## Escalation instead of confrontation

Every technique covered so far in this group — direct asks, role-play framing, payload splitting, guardrail tricks — shares an assumption: the disallowed request gets delivered, in some form, in a single message. Crescendo drops that assumption entirely. Instead of one message designed to slip past the model's defenses, an attacker opens with a request so mild it would never trigger a refusal — an innocuous factual question on a sensitive-adjacent topic — then nudges the conversation slightly further, turn by turn, in the same direction. Each step is only a small escalation from the one before it. Read against the step immediately prior, each one looks like a reasonable continuation, not a new disallowed request. Ten or fifteen turns later, the conversation has arrived somewhere the model would have refused outright had the final question been the opening line.

## Why consistency becomes a liability

Crescendo works because of a property that's usually a feature, not a bug: language models weigh the current response against the pattern established earlier in the conversation, favoring continuity with what they've already said. A model that just spent ten turns engaging substantively with a topic, getting progressively more specific at the user's request, feels internal pressure to continue that pattern rather than abruptly reverse and refuse — reversing would itself look like an inconsistent, discontinuous response given everything that came before. Safety training is shaped mostly by examples of single disallowed requests met with refusal. It's thin on examples of an entire multi-turn trajectory drifting somewhere it shouldn't, even though no individual step looks like a violation. A defense evaluating each turn independently — the way a content filter checking one message at a time naturally does — has no way to see the trajectory at all.

## Hiding content instead of pacing it

Encoding and obfuscation attacks solve a related problem through a different mechanism. Instead of pacing the request across turns, they disguise its surface form within a single message. A phrase a keyword filter would catch verbatim gets transformed so the literal flagged string never appears in the text the filter scans, while the underlying meaning survives well enough for the model to reconstruct it. Four routes do this: encoding the request in `base64`, so a filter scanning for plain-English trigger words sees only unreadable characters; leetspeak-style character substitution, replacing letters with visually or phonetically similar characters to defeat exact string matching while staying legible to the model (and usually a human); translating the request into a language the safety system has been less thoroughly trained against, exploiting uneven safety coverage across languages rather than any property of the encoding itself; and splitting a flagged word across extra whitespace, punctuation, or separate tokens, which the model's tokenizer and comprehension can often reassemble even though a simple string-matching pass never will.

## The same blind spot, two different exploits

What ties Crescendo to encoding attacks together is the actual lesson, not just a list of two techniques: both exploit safety evaluation happening on the wrong unit of analysis. A filter checking one message's literal surface text, in isolation from the conversation it sits inside, is structurally blind to intent that only resolves once several turns or several transformed fragments are combined. Crescendo spreads intent across time; encoding spreads it across representation. Neither needs to defeat the model's underlying judgment the way a role-play jailbreak does — both succeed purely by denying the surrounding safety machinery the information it needs to recognize what's happening. The practical implication: evaluating safety one message at a time, on raw surface text, is a fundamentally incomplete strategy. Real coverage requires looking at full conversational trajectories and at a message's reconstructed meaning, not just the literal bytes a single turn contains.
