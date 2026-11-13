---
title: Tokens and context
description: How text becomes numbers a model can process, and why the context window is a hard budget on everything it can see at once.
roadmap: prompt-engineering
node: tokens-and-context
group: foundations
video:
  provider: youtube
  id: "4A_nfXyBD08"
  start: 0
  end: null
creator:
  name: Annie Sexton
  channel_url: https://www.youtube.com/@AnnieSexton1
duration_seconds: 434
level: beginner
prerequisites: [how-llms-work]
takeaways:
  - Models operate on tokens, not words or characters, and token count is usually higher than word count for English text and far higher for code, rare words, or non-English languages.
  - The context window is the maximum number of tokens — input plus output combined — a model can process in a single call, and anything that doesn't fit simply isn't seen.
  - A large context window doesn't mean every token in it gets equal attention; models reliably perform worse at using information buried in the middle of a very long input than information near the start or end.
check_questions:
  - q: Why does the same sentence sometimes cost more tokens in one language than another?
    a: Tokenizers are trained on a corpus with a particular mix of languages and code, and they allocate shorter, more efficient token sequences to patterns that appeared often in that corpus — typically English — so text in under-represented languages or scripts gets split into more, smaller tokens.
  - q: If a model has a 128K-token context window, what exactly is being budgeted?
    a: The combined total of every token in the input (your prompt, any retrieved documents, conversation history) and every token the model generates in its output — not just the prompt alone.
deeper:
  - label: "IBM Technology — What is a Context Window? Unlocking LLM Secrets"
    url: https://www.youtube.com/watch?v=-QVoIxEpFkM
related_formulas: []
---

## Text has to become numbers first

A model doesn't read letters or words — it operates on tokens, and a token
is a chunk of text mapped to a number the model's internal math can use.
Before any prediction happens, your entire prompt gets run through a
tokenizer that breaks it into these chunks using a learned vocabulary,
typically built with an algorithm called byte-pair encoding: start from
individual characters, and repeatedly merge the most frequently co-occurring
pairs into single tokens until you have a vocabulary of a fixed size
(commonly in the tens of thousands of entries). Common whole words like
"the" or "model" usually end up as single tokens. Rarer words, made-up
words, and most non-English text get split into multiple smaller pieces —
"tokenization" might become "token" + "ization," for instance.

This is why token count and word count diverge, sometimes significantly.
English prose runs roughly 1.3 tokens per word on average. Code, which
mixes symbols, whitespace, and identifiers in ways that don't match the
tokenizer's training data as cleanly, often costs more tokens per visible
character than prose does. Languages underrepresented in the tokenizer's
training corpus — many non-Latin scripts, for example — can cost two to four
times as many tokens for the same meaning as equivalent English text. This
isn't a cosmetic detail: since both cost and the context budget below are
measured in tokens, a prompt that looks short to you as a human reader can
be expensive or budget-consuming in ways that aren't obvious until you
actually count.

## The context window is a hard ceiling, not a soft guideline

Every model has a context window: a fixed maximum number of tokens it can
process in one call. Critically, this budget covers the *combined* total of
everything — your system instructions, the conversation history, any
documents you've retrieved and pasted in, and the response the model is
about to generate — not just your input prompt in isolation. If your input
alone consumes 90% of a model's context window, you've left very little
room for a lengthy, detailed response, and some APIs will simply truncate or
error rather than let the budget overflow.

Context windows vary enormously by model and have grown quickly: where
early widely-used models topped out around 4,000 or 8,000 tokens, current
frontier models commonly offer context windows from around 100,000 tokens up
to 1 million or more. A bigger window means you can paste in more
background — a longer document, more conversation history, more few-shot
examples — but it is not free in two separate ways. First, cost: most APIs
price by the token, so a 200,000-token input costs roughly 25 times what an
8,000-token input does, before the response is even generated. Second,
latency: processing more input tokens takes the model more time before it
can start producing output.

## A full context window isn't used evenly

The subtler problem with large context windows is that stuffing them full
doesn't guarantee the model will use everything inside equally well. Models
tend to pay the most reliable attention to information near the beginning
and the end of a long input, with a measurable dip in recall for facts
placed in the middle — an effect sometimes called "lost in the middle."
Practically, this means that if you're retrieving several documents and
concatenating them into a prompt, the order matters: burying the single most
relevant document in position six of ten is riskier than putting it first
or last, even though all ten documents technically fit inside the context
window with room to spare.

This also means "does it fit in the context window" and "will the model
actually use it well" are two different questions, and prompt engineering
has to answer both. A model with a million-token window can technically
accept an entire book, but that doesn't mean every fact in that book is
equally retrievable by the model when it generates a response — which is
exactly why techniques like retrieval (pulling in only the most relevant
excerpts rather than entire documents) remain valuable even as raw context
windows keep growing.

## What to carry forward

Treat tokens as the real unit of cost, length, and capacity you're working
with, not words or characters — when later lessons talk about keeping a
prompt concise or trimming an output, token count is what's actually being
managed. And treat the context window as a strict, shared budget across
everything in a single call: instructions, history, retrieved content, and
the response all draw from the same pool, and position within that pool
affects how reliably the model uses what you put there.
