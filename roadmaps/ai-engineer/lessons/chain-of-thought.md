---
title: Chain of thought
description: Getting a model to work before it answers, from step-by-step prompting to reasoning models that do it internally.
roadmap: ai-engineer
node: chain-of-thought
group: prompting
video:
  provider: youtube
  id: "xCRvOUykOX0"
  start: 0
  end: null
creator:
  name: Google for Developers
  channel_url: https://www.youtube.com/@GoogleDevelopers
duration_seconds: 806
level: intermediate
prerequisites: [few-shot-prompting]
takeaways:
  - Generating intermediate steps gives a model more forward passes to spend on a problem, which is why reasoning before answering improves accuracy on multi-step tasks.
  - Reasoning models are trained to do this internally, so instructing them to think step by step adds little and can interfere.
  - A chain of thought is a generated artefact, not an audit trail — a model can reach the right answer through reasoning that does not justify it.
check_questions:
  - q: Why does thinking step by step help at all, mechanically?
    a: Each token is one forward pass. Writing intermediate steps lets the model spend many passes on a hard problem and condition each step on the last, instead of forcing the whole answer into a single pass.
  - q: Can you trust a model's stated reasoning as an explanation of its answer?
    a: No. The explanation is generated text, plausible by construction, and research shows models reach answers for reasons their stated chain does not mention. Treat it as a working surface, not evidence.
deeper:
  - label: "Denny Zhou (Berkeley) — LLM Reasoning, from the author of the CoT and self-consistency papers"
    url: https://www.youtube.com/watch?v=QL-FS_Zcmyo
  - label: "LangChain — Understanding and effectively using reasoning models"
    url: https://www.youtube.com/watch?v=f0RbwrBcFmc
related_formulas: []
---

## Why intermediate steps help, mechanically

A model gets exactly one forward pass per output token, and each pass can
only do a bounded amount of computation — roughly proportional to the
network's depth. Ask for a final answer immediately, and a hard multi-step
problem has to get solved within that single bounded pass, with nowhere to
put intermediate results. It's like being told to solve a long multiplication
problem entirely in your head, in one glance, with no scratch paper.

Ask the model to show its working first, and it gets as many passes as the
working is long. Each step's output becomes part of the input to the next
step, which is effectively a scratchpad it can read back from. That's the
entire mechanical explanation for why "think step by step" helps: the model
doesn't get smarter, it just gets more computation to visibly spend on a hard
problem. It also explains the exception — on tasks that aren't actually
multi-step, like a simple lookup or a direct classification, there's no extra
computation worth spreading out. Asking for reasoning there just adds latency
and cost.

## Self-consistency, worked through

**Self-consistency** improves on a single chain of thought by sampling
several independent reasoning paths at a non-zero temperature and taking the
answer most paths agree on. Picture asking an arithmetic word problem five
times at temperature `0.7`. The model might reach the correct answer along
three different lines of reasoning, and two wrong answers via two different
mistakes. Majority voting returns the correct answer, because the paths that
reach it agree with each other despite differing in wording, while the wrong
answers each arrived by an independent mistake and don't reinforce one
another.

This works because there are usually many ways to go wrong and comparatively
few ways to go right, so errors scatter while correct reasoning converges. It
costs roughly the sampled count times as much as a single call, which is why
it stayed mostly a research technique rather than a default production
pattern.

## Reasoning models: training replaces prompting

The biggest change since chain-of-thought prompting was introduced: some
current models are trained with reinforcement learning specifically to
produce extended internal reasoning before answering, rewarded whenever that
reasoning leads to a verifiably correct result. These models generate their
own internal deliberation, often lengthy, before committing to a final
answer, and usually expose a way to control how much computation to spend on
it — a reasoning-effort setting rather than a prompt instruction. For these
models, adding "think step by step" to the prompt is at best redundant, since
the behavior is already trained in, and it can occasionally interfere by
competing with the model's own learned strategy for structuring its
reasoning.

## Extensions beyond a single chain

Tree-of-thought and similar techniques generalize the idea further: explore
several reasoning branches and evaluate or backtrack between them, rather
than committing to one linear chain. It's closer in spirit to a search
algorithm than to a single stream of thought. These see use mainly where a
problem has a cheap way to check whether a branch is promising partway
through, since exploration cost grows quickly without that.

## Never trust the chain as an explanation

The strongest caution this lesson carries: generated reasoning reads like an
explanation, but it isn't reliably one. Research probing models that produce
a chain of thought has found cases where the model reaches an answer for
reasons its own stated reasoning never mentions. The chain is a plausible
narrative generated alongside the answer, not necessarily an accurate account
of how the answer was actually reached internally. If a user-facing product
needs to justify an answer, the retrieved sources and tool results behind it
are the real evidence. The model's narrated reasoning is, at best, a helpful
hint — never present it as proof.
