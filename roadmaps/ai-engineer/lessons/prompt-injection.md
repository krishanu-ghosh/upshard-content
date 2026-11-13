---
title: Prompt injection
description: Why instructions and data share one channel, direct and indirect injection, the lethal trifecta, and why no complete fix exists.
roadmap: ai-engineer
node: prompt-injection
group: safety
video:
  provider: youtube
  id: "rAEqP9VEhe8"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 742
level: intermediate
prerequisites: [tool-calling]
takeaways:
  - A model receives instructions and data in the same channel with no mechanism to tell them apart, which is why injection is a design property rather than a bug.
  - Indirect injection arrives through content the system retrieves — a web page, a document, a tool result — so the attacker never needs to talk to your application.
  - The lethal trifecta is private data, exposure to untrusted content, and a way to communicate outward; a system with all three can be made to exfiltrate.
check_questions:
  - q: Why can prompt injection not be fixed by better instructions in the system prompt?
    a: Your instructions and the attacker's arrive as the same kind of thing — tokens in one context. Telling the model to ignore malicious instructions is itself just more text, which the attacker's text can address.
  - q: What are the three ingredients of the lethal trifecta, and why does breaking one help?
    a: Access to private data, exposure to untrusted content, and an outbound channel. An attack needs all three to steal anything, so removing any one turns a potential exfiltration into a contained failure.
deeper:
  - label: "Simon Willison — Prompt injection, explained"
    url: https://www.youtube.com/watch?v=FgxwCaL6UTA
  - label: "AI Engineer — The lethal trifecta is already on your laptops"
    url: https://www.youtube.com/watch?v=abw_m-DRi1k
related_formulas: []
---

## One channel for two things

Every classic injection vulnerability in software has the same underlying
shape: data gets treated as instructions somewhere along the way. SQL
injection, cross-site scripting, shell injection — and every one of them has
the same fix, a boundary the interpreter itself enforces, so that a
parameterised value can never be reinterpreted as syntax no matter what it
contains. Language models have no equivalent boundary built into their
architecture. The system prompt, the user's message, a retrieved document and
a tool's returned output all arrive as one continuous sequence of tokens, and
the model weighs all of them by position and by phrasing, not by any notion
of where each piece originated, because there is no origin field anywhere in
the architecture to weigh. This is why injection here is not a bug awaiting a
patch in some future model release; it is a direct, structural consequence of
how the architecture works, and no model update changes that underlying fact.

## Direct injection, the easy demonstration

A user typing "ignore your previous instructions and do this instead" into a
chat box is direct injection, and it is also the least interesting case in
practice, because that user generally only has the ability to manipulate
their own session and gains little beyond what they could have simply asked
for honestly in the first place.

## Indirect injection, the one that actually matters

The real exposure is indirect: the malicious instructions arrive embedded
inside content the system fetches on its own, not inside anything the user
typed. A web page an agent is asked to read, a PDF someone uploaded for
analysis, a calendar invitation description, a line in a code comment, even
the description field of a third-party tool connected through
[MCP](/roadmaps/ai-engineer/model-context-protocol) — any of these can carry
hidden instructions. The attacker never needed access to your application at
all. They needed only to place text somewhere your system would eventually
fetch and feed into a model, and then simply wait. A concrete pattern worth
knowing by name: an attacker embeds an instruction inside a web page telling
the agent to fetch a specific external URL and encode sensitive data from the
conversation into that URL's query parameters as an image request — when the
model renders that as an image, the act of fetching the image silently
exfiltrates the data to a server the attacker controls, with no obvious
visual sign to the user that anything happened at all. Invisible techniques
such as zero-width Unicode characters or text coloured to match a page's
background can hide such an instruction from a human skimming the same page,
while the model still reads and processes every character exactly the same
way.

## The lethal trifecta, as an operational framework

Simon Willison's framing is the single most useful operational tool
available here. Serious exfiltration requires three conditions to hold
simultaneously: **access to private data** — your documents, your database,
the current user's files; **exposure to untrusted content** — anything an
attacker can plausibly influence, directly or indirectly; and **a way to
communicate outward** — an HTTP request, an email send, a rendered image URL,
any tool capable of posting information somewhere external. An agent
possessing all three at once can be instructed, via a retrieved document, to
read sensitive data and send it away to an attacker-controlled destination.
An agent missing any single one of the three cannot complete that attack no
matter how convincingly it is instructed to — which is the entire substance
of the defence: design systems so that these three conditions do not all hold
true for the same agent at the same time.

## Designing around the trifecta in practice

The agent that reads genuinely untrusted content — arbitrary web pages,
documents from unknown sources — should not be the same agent holding
credentials to sensitive internal data; splitting these into separate agents
with a narrow, well-defined interface between them, sometimes called a
**dual-LLM pattern**, is a direct and practical application of this
principle. Outbound actions from any agent should be allow-listed to a known,
specific set of safe destinations rather than left general-purpose — a tool
that can fetch an arbitrary URL supplied by the model is itself an
exfiltration channel, however innocuous and useful it looks in the demo where
it was first added. Anything irreversible that the agent might do — sending a
message, making a payment, deleting a record, publishing content — should
require explicit human approval, and critically, that approval screen must
show the real, concrete action about to be taken rather than a summary
sentence the model itself wrote describing what it intends to do, since a
compromised model could write a reassuring summary for a harmful action just
as easily as an honest one.

## Why a secret system prompt is not a security boundary

A common but mistaken instinct is treating the system prompt's contents as
confidential and relying on instructing the model not to reveal them as a
meaningful defence. This is not a security boundary at all: the system prompt
and the untrusted content both exist in the same undifferentiated channel
described above, so an attacker who can get the model to follow their
embedded instructions can generally also get it to reveal whatever it was
told not to reveal, through the same mechanism. Filters and classifiers that
scan input and output for suspicious patterns help at the margin and should
be deployed, covered further in
[guardrails and moderation](/roadmaps/ai-engineer/guardrails-and-moderation),
but they are a speed bump rather than a wall — the architecture itself, not
any filter layered on top of it, is what actually has to bound the damage
when, not if, something gets past them.
