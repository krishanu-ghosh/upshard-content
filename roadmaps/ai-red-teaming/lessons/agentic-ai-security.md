---
title: Agentic AI security and excessive agency
description: When an injected instruction can make an autonomous agent act, not just answer, one compromised integration can cascade into real-world harm.
roadmap: ai-red-teaming
node: agentic-ai-security
group: system-attacks
video:
  provider: youtube
  id: "TWhKGqYQT9g"
  start: 0
  end: null
creator:
  name: Embrace The Red
  channel_url: https://www.youtube.com/@embracethered
duration_seconds: 2700
level: advanced
prerequisites: [rag-and-tool-attack-surface, multi-turn-and-encoding-attacks]
takeaways:
  - Excessive agency means an agent holds more autonomy, reach, or permissions than its actual task requires, so a single successful injection can exercise far more capability than the task ever needed.
  - Cascading tool misuse happens when one injected instruction causes an agent to chain several of its own legitimate tool calls together into an outcome none of those calls would cause individually.
  - "A coding or computer-use agent that is compromised through injection becomes the attacker's remote hands on the victim's own machine, a pattern researcher Johann Rehberger calls a \"ZombAI\"."
  - Agent-to-agent lateral movement, demonstrated concretely against Gemini via a poisoned calendar invite, shows that a compromised agentic surface in one system can reach into an entirely separate integration.
check_questions:
  - q: What does "excessive agency" mean in the context of an AI agent, and why does it matter for security?
    a: It means the agent has been granted more autonomy, permissions, or reach than the specific task it's performing actually needs, so if an attacker manages to inject even one instruction, the agent has far more capability available to misuse than a narrowly scoped agent would.
  - q: What is "cascading tool misuse," and how does it differ from a single malicious tool call?
    a: It's when one injected instruction causes an agent to chain together several of its own individually legitimate tool calls — for example, read a file, then compress it, then send it somewhere — into a harmful outcome that no single one of those calls would have caused on its own.
  - q: What did the Gemini "Invoking Agents with a Google Calendar Invite" research demonstrate about agentic risk?
    a: That a prompt-injection payload hidden in something as mundane as a calendar invite description could reach an agent and get it to take actions in a completely separate connected system, showing that a compromised agentic surface in one integration can move laterally into another.
  - q: Why does "the agent can act, not just answer" make these risks qualitatively worse than the prompt-only attacks covered earlier in this roadmap?
    a: A manipulated chatbot that only produces text can, at worst, mislead or offend the person reading it, but an agent that is manipulated into acting can send real emails, move real money, modify real files, or run real code, turning a successful injection into a real-world consequence rather than a bad answer.
deeper:
  - label: "DEF CON 33 - Invoking Gemini Agents with a Google Calendar Invite — Ben Nassi, Or Yair, Stav Cohen"
    url: https://www.youtube.com/watch?v=CUxbDRR0A8I
related_formulas: []
---

## Acting is the risk multiplier

Every attack covered earlier in this roadmap's system-attacks group assumed
a model that reads untrusted content. Agentic systems add a second
dimension: the model doesn't just read, it acts, often across multiple steps
and multiple tools, with no human reviewing each individual step before it
happens. A chat assistant that gets successfully injected can, at worst,
produce a bad or embarrassing answer. A coding agent, a computer-use agent,
or an autonomous assistant wired into real tools that gets successfully
injected can run commands, install software, send messages, and touch files
on a real machine. The underlying injection techniques are the same ones
already covered — indirect prompt injection through content the agent reads
— but the consequence changes entirely once the compromised model has hands,
not just a mouth.

## Excessive agency: more capability than the task needs

The first concept this lesson introduces by name is excessive agency: an
agent configured with more autonomy, more reach, or more permissions than its
actual task requires. A coding assistant that only needs to read a
repository and suggest edits, but is also wired up with unrestricted shell
access and outbound network access, is carrying excessive agency relative to
its job — and every one of those extra capabilities is also available to
whatever injected instruction manages to reach the model, through a
malicious comment in a file it's asked to review, a poisoned dependency's
README, or a crafted issue description. The fix isn't exotic: scope an
agent's permissions to what its task genuinely requires, and treat every
additional capability granted beyond that as additional attack surface, not
as harmless headroom.

## Cascading tool misuse: legitimate steps, illegitimate outcome

The second concept is cascading tool misuse, and it's more subtle because no
single step looks wrong. An agent with access to a file-reading tool, a
compression tool, and a network-upload tool has three individually
reasonable capabilities. One injected instruction, hidden in a document the
agent was asked to summarize, can direct it to read a sensitive file,
compress it, and upload the result — three calls, each one a completely
normal use of a completely legitimate tool, that together constitute data
exfiltration nobody authorized. This is why reviewing an agent's available
tools in isolation is insufficient; the dangerous question is what
combinations of its own legitimate capabilities an attacker could chain
together, because an agent built to chain tool calls autonomously will do
exactly that with an attacker's goal in mind as readily as with the user's.

## ZombAIs: the agent as remote hands

Johann Rehberger's more recent research, the basis for this lesson's video,
pushes this from a design concern into a demonstrated outcome against
coding and computer-use agents such as Claude Code, GitHub Copilot, and
Devin-style autonomous assistants: an injected instruction convincing an
agent to download and run attacker-supplied malware, turning that agent into
what he terms a "ZombAI" — a compromised agent acting as the attacker's
remote hands on the victim's own machine. The point isn't that any one
product is uniquely flawed; it's that any sufficiently capable, sufficiently
autonomous coding or operating agent is, by design, a general-purpose remote
execution capability, and an injection that reaches it inherits that
capability wholesale.

## Lateral movement across agentic surfaces

The DEF CON 33 Gemini research in this lesson's deeper link extends the
pattern one step further: a prompt-injection payload hidden inside a Google
Calendar invite's description reached a Gemini-integrated agent and caused
it to take action in an entirely separate connected system. The entry point
was almost comically mundane — accepting a calendar invite is not an action
most people think of as security-sensitive — but because the agent's
reasoning loop treated the invite's text as input to act on, one compromised
integration became a path into another. This is the clearest illustration of
why agentic risk is qualitatively different from the prompt-only attacks
earlier in this roadmap: once a system can act, and act across multiple
connected surfaces, "the model read something malicious" and "something
harmful happened somewhere else entirely" become the same event.
