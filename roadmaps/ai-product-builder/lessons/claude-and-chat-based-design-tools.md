---
title: Claude and Chat-Based Design Tools
description: Designing UI and UX through back-and-forth chat with Claude or a similar model, from a product requirements doc to a prompt-ready prototype.
roadmap: ai-product-builder
node: claude-and-chat-based-design-tools
group: prototyping-with-ai
video:
  provider: youtube
  id: "nDHXLnwlIaY"
  start: 0
  end: null
creator:
  name: Sean Kochel
  channel_url: https://www.youtube.com/@iamseankochel
duration_seconds: 1087
level: intermediate
prerequisites:
  - choosing-a-prototyping-tool
takeaways:
  - "A chat-based design workflow replaces drag-and-drop with a structured conversation: write a product requirements doc, refine a UX definition, then prompt toward a prototype."
  - "Writing the PRD in chat first forces decisions about scope and user flow before any screen gets built, which is cheaper to fix in text than in a generated component."
  - "A specific 'Claude Skill' referenced in some tutorials is just a packaged version of this same method — you can run the underlying process manually without owning that file."
  - "Iterating on the UX definition before touching code means later prompts to an app generator are shorter and more precise, because the ambiguity got resolved earlier."
  - "The output of this phase is a spec plus a direction, which you then feed into a prototyping tool rather than a finished app in itself."
check_questions:
  - q: What are the three stages of the chat-based design workflow described in this lesson?
    a: Write a product requirements doc through conversation, iterate on a UX definition built from that doc, then prompt toward a working prototype using that definition as the brief.
  - q: If you don't have the specific Claude Skill a tutorial references, can you still follow the workflow?
    a: "Yes — the skill is just a packaged shortcut for a method you can run manually: ask Claude to help draft a PRD, refine the UX description through follow-up turns, and then use that refined description as your prompt to a builder tool."
  - q: Why write the PRD and UX definition before generating any screens?
    a: Resolving ambiguity in plain text is far cheaper than resolving it after code exists — a flawed user flow caught in a document takes a sentence to fix, while the same flaw discovered in a generated prototype means re-prompting or manually reworking components.
deeper:
  - label: "Building a Dashboard in Lovable: Claude AI Mockups, Figma to Lovable & Framer Landing Page"
    url: "https://www.youtube.com/watch?v=SL0GDHkQX9A"
related_formulas: []
---

## Designing by talking instead of dragging

Most prototyping tools assume you'll either type one big prompt or drag components around a canvas. There's a third approach that fits naturally between those two: a sustained conversation with Claude or a similar model, where the design itself emerges through back-and-forth before any UI gets generated. Instead of trying to compress an entire product idea into a single perfect prompt, you build it up in stages — first establishing what the product needs to do, then what the experience should feel like, and only then asking for a prototype. This lesson's embedded video demonstrates this using a custom "Claude Skill," a packaged set of instructions the creator built for repeatable use. You won't have that exact file, and you don't need it — the value is the underlying method, which this lesson walks through directly so it stands on its own.

## Stage one: writing the PRD in chat

The first stage is drafting a product requirements document the same way you'd write one with a colleague, except the colleague is a model that can immediately push back with clarifying questions. You describe the problem you're solving, who it's for, and what success looks like; the model asks about edge cases, missing user types, or unstated assumptions you hadn't considered. The output isn't meant to be a polished artifact — it's a working document that captures scope and intent in plain language, before a single screen exists. Catching a confused user flow or a missing permission level at this stage costs you a sentence of revision. Catching the same problem after a prototyping tool has already generated components around the wrong assumption costs you a rewrite.

## Stage two: turning the PRD into a UX definition

Once the PRD is solid, the next round of conversation narrows it toward something closer to a design brief: what screens exist, what the primary flow through them looks like, what state each screen needs to track, and where the experience should feel fast versus where it's fine to ask the user to wait. This is still entirely text — no mockups, no wireframes — but it's text with enough specificity that a prototyping tool could act on it directly. The reason to do this as its own pass, rather than folding it into the PRD, is that product requirements and UX decisions get revised independently; mixing them makes both harder to iterate on cleanly.

## Stage three: prompting toward a prototype

With a refined UX definition in hand, the actual prompt you send to Lovable, Bolt, v0, or whatever generator you've chosen becomes dramatically more precise, because most of the ambiguity a vague one-shot prompt would have forced the model to guess at has already been resolved in conversation. You're no longer asking the builder tool to invent a user flow; you're asking it to implement one you've already specified. This tends to produce fewer wasted iterations and output that's closer to what you actually wanted on the first or second pass, because the hard thinking happened earlier, in a medium — plain text — that's cheap to revise.

## Why this complements, not replaces, prompt-to-app tools

This chat-based process isn't a competing prototyping tool; it's a design phase that happens before you touch one. The PRD-then-UX-then-prompt sequence works whether you ultimately generate the app in Lovable, build it by hand, or hand the spec to a developer. What makes it worth adopting is that the cost of changing your mind drops enormously while you're still in conversation, compared to after code — generated or otherwise — already exists. Treat the chat transcript itself as a disposable intermediate artifact: what matters is the clarity it leaves you with by the time you actually start building.
