---
title: How LLMs work
description: Next-token prediction, a learned function with billions of parameters, and why the output is a probability distribution rather than a lookup.
roadmap: ai-engineer
node: how-llms-work
group: foundations
video:
  provider: youtube
  id: "LPZh9BOjkQs"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 478
level: beginner
prerequisites: [what-is-an-ai-engineer]
takeaways:
  - A language model is one enormous learned function that maps a sequence of tokens to a probability distribution over the next token.
  - Generation is that function run repeatedly, each output appended to the input — which is why a long response costs far more than a long prompt.
  - Training sets the parameters so real text gets assigned high probability. Nothing in the model stores text to look up later.
check_questions:
  - q: Why is a model's output a distribution rather than a single answer?
    a: The final layer scores every token in the vocabulary, and a softmax turns those scores into probabilities. Picking one is a separate decision made by the sampler, not by the model.
  - q: Why does generating 1,000 tokens cost more than reading 1,000 tokens?
    a: The input is processed in one parallel pass, but each output token needs its own forward pass through the whole network, conditioned on every previous token.
deeper:
  - label: "3Blue1Brown — Transformers, the tech behind LLMs"
    url: https://www.youtube.com/watch?v=wjZofJX0v4M
related_formulas: [softmax, cross-entropy, perplexity]
---

## The one idea worth holding on to

A language model predicts the next token. That's it. Chat, reasoning, tool
use, agents — all of that is this one operation wrapped in enough scaffolding
to look like something else. It's the same move autocomplete makes on your
phone, just run by a vastly bigger function trained on vastly more text.

Keeping this in view explains behavior that otherwise looks baffling: why a
model invents a citation with total confidence, why asking it to think step
by step changes the answer, why the same prompt gives a different result
twice.

## From text to a distribution

The model's input is a sequence of tokens — think of a token as roughly a
word fragment; the next lesson covers this properly. Each token gets
converted to a vector by a lookup table called an embedding layer, so the
sequence becomes a stack of vectors.

Those vectors pass through dozens of repeated blocks. Each block does two
things: **self-attention**, where every position pulls information from every
earlier position, weighted by how relevant that earlier position is to this
one; and a **feed-forward network**, which processes each position's vector
on its own and is where most of the model's raw parameter count actually
lives.

After the last block, a final linear layer projects the last position's
vector onto one score per word in the vocabulary — tens of thousands of
scores. A softmax turns those scores into a probability distribution that
sums to one. That distribution is the model's entire output for this step.
Generating one token means sampling from it. Generating a whole response
means repeating this, once per output token, with the new token appended to
the input each time.

## Where the parameters came from

Those billions of parameters aren't programmed. They're fit by gradient
descent during **pretraining**, against one simple objective: given
everything so far, predict the next token correctly, scored against
trillions of tokens of real text. Nothing in that objective asks the model to
be honest, helpful, or careful — only to predict plausible continuations.

The behavior that looks like following instructions or refusing harmful
requests gets added afterward, in a second stage usually called
**instruction tuning** or **alignment**. Here the model trains further on
examples of the behavior a product wants, often refined again using human or
automated feedback on which of several responses is better. The pretrained
model is a raw continuation engine. The model you call through an API has had
two or three more rounds of training stacked on top to make it act like an
assistant.

## Why generation is the expensive half

Processing the prompt happens in a single pass — every position attends to
every earlier position at once, which parallelizes beautifully on a GPU.
Producing the response can't be parallelized that way, because token five
has to exist before token six can be generated. Each output token is its own
full pass through every layer, conditioned on everything produced so far.

This asymmetry is the direct cause of the pricing split covered in
[cost and latency](/roadmaps/ai-engineer/cost-and-latency): reading is cheap
because it's parallel, writing is expensive because it's sequential.

## What "no memory" actually means

A deployed model holds no state between API calls. Every turn of what feels
like a continuous conversation is, mechanically, a brand-new forward pass
over the entire transcript, re-sent from scratch. The model isn't
"remembering" earlier turns — it's reading them again, exactly as if for the
first time, packed into the same context window as the current question.

That's why a conversation that has grown very long costs more per turn even
when the new message is short. It's also why the context window has to be
managed as a budget rather than treated as free memory — the subject of a
later lesson in this group.
