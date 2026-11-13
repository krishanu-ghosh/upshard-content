---
title: Provider-native agent SDKs
description: "Model providers now ship their own first-party agent tooling — the fastest path to a working agent, at the price of being locked to that provider."
roadmap: ai-agents
node: provider-native-agent-sdks
group: frameworks
video:
  provider: youtube
  id: "tK32trvj_b4"
  start: 0
  end: null
creator:
  name: OpenAI
  channel_url: https://www.youtube.com/@OpenAI
duration_seconds: 2860
level: intermediate
prerequisites: [native-function-calling, mcp-vs-custom-integrations]
takeaways:
  - A provider-native agent SDK — OpenAI's Agents SDK or Assistants API, Google's Gemini agent tooling — is built and maintained by the same company that built the model, which means tighter integration with that model's own function-calling, tool-use, and context-handling behavior than any third-party framework can guarantee.
  - "That tight integration is also a form of lock-in: code written against a provider's own agent primitives generally does not port to a different provider without real rework, because the abstractions themselves are shaped around that one provider's API."
  - A provider-agnostic framework like LangGraph trades some of that first-party polish for the ability to swap the underlying model provider with comparatively little rework.
  - The right choice depends on how likely a provider switch actually is for a given project, not on which option is more sophisticated in the abstract.
check_questions:
  - q: Why can a provider-native agent SDK often get a working agent running faster than a third-party framework?
    a: It's built by the same team that built the model and its API, so it's tuned to that model's specific function-calling and tool-use behavior with no translation layer in between, which removes a whole category of integration friction a general-purpose framework has to handle more generically.
  - q: What's the real cost of building on a provider-native SDK that a provider-agnostic framework avoids?
    a: The agent's logic ends up expressed in terms of that one provider's specific primitives, so switching to a different model provider later means re-architecting those parts of the agent rather than mostly swapping a configuration value, which a provider-agnostic framework is built to avoid.
deeper:
  - label: "Build agents with Gemini API — Google for Developers"
    url: https://www.youtube.com/watch?v=OdrOmc_RX8A
  - label: "How to use Gemini function calling with Cloud Run — Google Cloud Tech"
    url: https://www.youtube.com/watch?v=xC4PrDBrQ5I
related_formulas: []
---

## A new category: the provider builds the framework

For a while, if you wanted to build an agent, you reached for a third-party
framework — something written by a separate team, designed to work across whichever
model you plugged into it. That's no longer the only option. OpenAI ships its own
Agents SDK (and before it, the Assistants API) as a first-party way to build agents
against its own models. Google ships its own agent tooling built around Gemini's
function-calling and tool-use capabilities. These aren't community frameworks that
happen to support a given provider well — they're built by the same organization that
trains the model and controls its API, specifically to be the most direct path to an
agent running on that model.

## Why first-party integration is a genuine advantage

A third-party framework has to work across multiple providers, which means it
necessarily abstracts over differences between them — in how each provider formats a
tool call, how each handles multi-turn tool use, what each one returns when a call
fails or needs clarification. That abstraction is valuable, but it's also a layer
that can lag behind a provider's own newest capabilities, or smooth over a difference
in a way that loses some provider-specific behavior. A provider-native SDK has no
such layer to maintain: when the Agents SDK handles OpenAI's own tool-calling
conventions, it's talking to the API exactly as its own team understands it, often
including features or behaviors that haven't yet been generalized into third-party
frameworks at all. For someone committed to one provider who wants the agent working
as fast as possible with the least friction, this is a real, not merely marketed,
advantage.

## Why that advantage comes with a cost

The same tight fit that makes a provider-native SDK fast to build with is what locks
you into it. The SDK's core abstractions — how it represents a tool, how it threads
conversation state, how it structures a multi-step agent run — are shaped around that
one provider's specific API design, not around some generic model of what an agent
is. Code built this way doesn't translate to a different provider's SDK by changing a
configuration value. The concepts underneath often don't line up one-to-one, so
porting means re-architecting the agent's core logic, not swapping an endpoint.

## A worked scenario

Imagine a startup building a customer-support agent on OpenAI's Agents SDK because
it's the fastest way to ship a first version, and it is — the team has a working agent
in a fraction of the time a general-purpose framework would have taken, because the
SDK's tool-calling and state-handling line up exactly with the model they're already
calling. Eighteen months later, a new model from a different provider offers
meaningfully better quality or pricing for their use case, and the team wants to
switch. If they'd built on a provider-agnostic framework like LangGraph, switching
providers would mostly mean changing which model the graph calls and retesting — the
graph's nodes, edges, and state don't know or care which provider sits behind them.
Having built directly on OpenAI's own SDK, the team instead has to re-express large
parts of the agent's structure in terms of the new provider's primitives, because the
original code was never written in provider-agnostic terms to begin with. Neither
choice was wrong at the time — the provider-native SDK genuinely got them to market
faster — but the tradeoff it carried has now come due.

## Making the call

This is a real tradeoff, not a question of which option is more advanced. If a
project is firmly committed to one provider for the foreseeable future — because of
cost, a specific capability, or an existing relationship — a provider-native SDK is
often the fastest, best-supported way to build the agent, and worrying about
portability you won't need is wasted effort. If switching providers is a real
possibility, whether to chase quality, pricing, or avoid dependence on one company,
a provider-agnostic framework is worth the modest extra effort up front, because the
cost of lock-in is paid later, all at once, and usually at a less convenient time than
now.
