---
title: The RAG and tool-calling attack surface
description: In RAG and tool-calling systems, retrieved documents and tool outputs are untrusted input channels, not just the user's own prompt.
roadmap: ai-red-teaming
node: rag-and-tool-attack-surface
group: system-attacks
video:
  provider: youtube
  id: "ADHAokjniE4"
  start: 0
  end: null
creator:
  name: Ekoparty Security Conference
  channel_url: https://www.youtube.com/@EkopartyConference
duration_seconds: 2700
level: intermediate
prerequisites: [prompt-injection-direct-and-indirect]
takeaways:
  - In a RAG system, any document that can end up in the retrieval corpus is a potential injection point, and the attacker never has to interact with the chat at all — they only need their content indexed.
  - A tool's return value is as much an untrusted input channel as its arguments; a browsing tool's fetched page or a calendar tool's retrieved invite can carry instructions the model will read as if they came from the user.
  - A tool that can both read untrusted content and take a consequential action, such as sending email or making a purchase, combines an injection point and a payload in a single component, which is why scoping a tool's permissions matters as much as scoping its inputs.
  - Indirect injection through RAG and tools scales naturally, because poisoning one widely retrieved document or one commonly used tool integration can affect every user session that touches it afterward.
check_questions:
  - q: How does a RAG-based injection differ from the attacker typing a malicious instruction directly into the chat?
    a: The attacker never touches the chat session at all; they plant or compromise a document that the system's retrieval step later fetches and feeds to the model as trusted context, so the injection rides in through the corpus rather than through user input.
  - q: Why is a tool's return value considered an attack surface, not just its input parameters?
    a: Because the model reads whatever a tool hands back — a web page's contents, an email body, a calendar invite's description — as context to reason over, and nothing inherently distinguishes attacker-planted text inside that returned content from the tool's legitimate output.
  - q: What makes a tool "over-permissioned" in a way that is specifically dangerous for injection attacks?
    a: A tool that reads untrusted external content and can also take a consequential, real-world action puts the injection point and the payload's execution path in the same component, so a single instruction hidden in content the tool reads can directly trigger that action with no separate approval step.
deeper:
  - label: "Real-world exploits and mitigations in LLM applications (37c3) — Johann Rehberger / Embrace The Red"
    url: https://www.youtube.com/watch?v=qyTSOSDEC5M
related_formulas: []
---

## From one hidden instruction to a whole architecture

The earlier lesson on direct and indirect prompt injection established the core idea: a model can't reliably tell a developer's instructions apart from instructions that merely arrive in its input, so text an attacker controls can get treated as a command. That lesson's indirect example — a hidden instruction buried in a document the model is asked to read — was a single case. Once a system is built around retrieval-augmented generation or tool-calling, that single case becomes a structural property of the whole architecture. Both patterns are, at bottom, mechanisms for feeding the model text that came from neither the user nor the developer. Johann Rehberger's Ekoparty talk was among the first to lay this out systematically across real, shipped LLM products — Bing Chat, ChatGPT's plugin ecosystem — showing live that an attacker who never opens the chat window can still get the model to act on their behalf, purely through content the model was asked to read or a tool was asked to fetch.

## RAG: the corpus itself is reachable

A retrieval-augmented system answers questions by pulling relevant passages
from a document store and handing them to the model as context, on the
assumption that whatever gets retrieved is trustworthy background
information. That assumption fails the moment any part of the corpus is
writable, scrapeable, or editable by someone outside the organization's
control — a public wiki a company's support bot indexes, a user-submitted
product review, a webpage the pipeline crawls. An attacker doesn't need
access to the chat interface or even to know who will ask what; they need
only get a passage containing a hidden instruction into the index, phrased to
look like ordinary content to a human skimming it, and wait. The next user
whose query happens to retrieve that passage inherits the injection, with no
interaction between the attacker and that victim at any point. This is a
fundamentally different threat model from a chat-based attack: it's closer
to watering-hole poisoning than to direct conversation, and it scales with
however many users' queries end up retrieving the poisoned document.

## Tool outputs: the return value is also an input

Tool-calling introduces the same structural problem from a different angle.
Developers naturally focus on validating what arguments a model passes into
a tool call, but the more dangerous channel often runs the other direction:
whatever the tool hands back becomes part of the model's context exactly as
if it had been typed by the user. A browsing tool that fetches a webpage on
the model's behalf is retrieving content the page's author controls
entirely; nothing stops that page from containing text aimed squarely at the
model reading it rather than at a human visitor. A calendar or email tool
that retrieves an invite or a message is reading content any sender could
have crafted. In each case, the tool did exactly its job and returned exactly
what it fetched — the vulnerability is that the calling model has no reliable
way to mark that returned content as "data to summarize" rather than
"instructions to follow."

## Why over-permissioned tools turn an injection into damage

An injection that only makes a model say something strange is an annoyance.
An injection that makes a model act is the real risk, and that risk
concentrates wherever a single tool combines two properties: it ingests
untrusted content, and it can independently take a consequential action, such
as sending an email, writing a file, or making a purchase. A tool shaped that
way hands an attacker both the injection point and the payload's execution
mechanism in one place, with no separate human approval between "the model
read something" and "the model did something because of it." Reducing this
risk means scoping tools narrowly — separating "read untrusted content" from
"take an irreversible action" wherever practical, and requiring confirmation
before the latter — rather than relying on the model to recognize an
injected instruction when it has already shown, repeatedly, that it often
can't.
