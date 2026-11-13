---
title: Building a red-team lab
description: Why practicing AI red teaming requires a dedicated, legal target, and a concrete checklist for standing up your own practice environment.
roadmap: ai-red-teaming
node: building-a-red-team-lab
group: methodology
video:
  provider: youtube
  id: "_yfiUQSbdPY"
  start: 0
  end: null
creator:
  name: NetworkChuck
  channel_url: https://www.youtube.com/@NetworkChuck
duration_seconds: 720
level: beginner
prerequisites: [llm-security-testing-tools]
takeaways:
  - "You cannot ethically or legally throw untested jailbreak attempts at a random production system just to practice, so a purpose-built lab is a prerequisite for building the skill, not an optional extra."
  - "Deliberately vulnerable LLM and agent targets, plus CTF-style challenges like Gandalf, give you a safe, repeatable place to fail and learn without harming anyone."
  - "A workable personal lab needs three things: a model you're allowed to attack, one or two of the open-source tools from the previous lesson installed, and a known benchmark challenge to calibrate yourself against."
  - Benchmarking against a well-known public challenge before testing anything real tells you honestly whether your current skill level is ready for an actual engagement.
check_questions:
  - q: Why shouldn't someone learning AI red teaming just practice against a live production chatbot they find online?
    a: Doing so is both unauthorized and potentially illegal under computer-fraud laws, and even when it technically isn't, it risks causing real harm or disruption to a system nobody asked you to test, which is why the field relies on purpose-built vulnerable targets for practice instead.
  - q: What are the three components of a minimal personal red-team practice setup?
    a: A local or sandboxed model or application to target, one or two open-source red-teaming tools installed and configured, and a known CTF-style challenge used as a benchmark to calibrate your own skill level.
  - q: What is Gandalf, and what role does a challenge like it play in learning this skill?
    a: Gandalf is a well-known public prompt-injection challenge where the goal is to extract a secret password from a model guarded by increasingly strong defenses, and it serves as a free, legal benchmark for testing and improving your own prompting and injection technique.
deeper:
  - label: "PyRIT Setup Workshop: AI Red Teaming on Windows — Null:404 Cyber Security"
    url: https://www.youtube.com/watch?v=lkCnueQhbss
related_formulas: []
---

## Skill without a target is just reading

Every lesson in this roadmap so far has explained a technique, a tool, or a methodology. None of that becomes a skill until you've actually done it against something that talks back. This is true of security work generally — nobody becomes a competent penetration tester by only reading about SQL injection — but it carries an extra constraint in AI red teaming that's worth stating plainly before going further.

## Why you need a dedicated target, not a convenient one

Unlike a classic web app pentest, where a tester might at least argue they're probing their own staging environment, the temptation in AI red teaming is to just try a jailbreak against whatever public chatbot is sitting in a browser tab. That's a bad idea for reasons that are both legal and ethical. Testing a system you don't have authorization to test is, depending on jurisdiction and the system's terms of service, potentially a violation of computer-fraud law — independent of whether you cause any actual harm.

And even where it's technically permitted, firing untested adversarial prompts at a live production system risks consequences you don't control: tripping abuse-detection systems that flag an innocent account, generating harmful content that gets logged and reviewed by someone with no idea it was a practice attempt, or simply wasting the resources of a system that owes you nothing. None of this is necessary. The field has produced an entire ecosystem of targets built specifically to be attacked, with no victim and no ambiguity about permission.

## What a good practice target looks like

A practice target should have three properties: it should be legal and explicitly intended to be attacked, it should be realistic enough that what you learn transfers to a real system, and it should give you some way to tell when you've succeeded. CTF-style challenges hit all three directly.

Gandalf, probably the best-known example, puts a model behind a password with escalating levels of defense and asks you to extract the secret through prompt injection — a clean, scored way to practice getting a model to reveal something it's been instructed to protect. Deliberately vulnerable LLM and agent labs go further, standing up an entire small application — a RAG pipeline, a tool-calling agent — with intentionally planted weaknesses. That lets you practice the full chain this roadmap covers: finding a prompt injection point, escalating it through a tool call, and chaining it into something consequential, exactly the pattern covered in the system-attacks and agentic-security lessons earlier in this roadmap. Auto-parts-style CTF scenarios and "agent breaker" labs exist precisely to give that end-to-end practice surface without ever touching a real company's system.

## A concrete starting checklist

Standing up a personal lab doesn't require much. First, get a target: either a small open-weight model you run locally (so there's no question of authorization, since it's entirely yours), or a known deliberately-vulnerable application built for this purpose. Second, install one or two of the tools from the previous lesson — `garak` is the easiest first install for a broad scan, and it's worth running a scan against your local model before attempting anything more elaborate. Third, pick a known CTF-style challenge, like Gandalf, and work through it before testing anything that matters.

That last step does real work beyond just being fun. A known public challenge with a known difficulty curve gives you an honest benchmark of where your current skill actually sits — valuable information before anyone, including you, decides you're ready to red-team something with real stakes attached.

## Practice compounds into the rest of the roadmap

Every technique from the prompt-attacks and model-attacks groups earlier in this roadmap is something you can and should actually try inside this kind of lab: a jailbreak framing against your local model, a prompt-injection payload against a vulnerable RAG demo, an automated `garak` sweep compared against your own manual attempt at the same target. Reading about Crescendo or Skeleton Key is not the same as watching one work — or more usefully, watching one fail and figuring out why. A personal lab is what turns the rest of this roadmap from reference material into practiced, demonstrable skill, and it's the last piece this methodology group leaves you with before the roadmap moves on to defense.
