---
title: Tokens and tokenization
description: Subword tokenization, byte-pair encoding, and why token counts rather than word counts are the unit of both cost and context.
roadmap: ai-engineer
node: tokens-and-tokenization
group: foundations
video:
  provider: youtube
  id: "4A_nfXyBD08"
  start: 0
  end: 374
creator:
  name: Annie Sexton
  channel_url: https://www.youtube.com/@AnnieSexton1
duration_seconds: 434
level: beginner
prerequisites: [how-llms-work]
takeaways:
  - A token is a subword fragment, not a word or a character, and byte-pair encoding builds the vocabulary by repeatedly merging the most frequent adjacent pair.
  - Token counts are the unit of billing and of the context window, and they do not track word counts in any stable way.
  - Tokenizers are trained mostly on English, so other languages, rare names, long numbers and unusual formatting consume disproportionately many tokens.
check_questions:
  - q: Why does subword tokenization exist rather than one token per word?
    a: A word-level vocabulary cannot represent anything it has not seen, and a character-level one makes sequences far too long. Subwords keep common words whole and still spell out anything unseen.
  - q: Why might the same sentence cost twice as many tokens in one language as in another?
    a: The merges were learned from a corpus that is mostly English, so text in a language the tokenizer saw less of falls back to shorter, more numerous fragments.
deeper:
  - label: "Vizuara — A visual introduction to tokenization in LLMs"
    url: https://www.youtube.com/watch?v=APnKbi448O4
  - label: "Andrej Karpathy — Let's build the GPT Tokenizer"
    url: https://www.youtube.com/watch?v=zduSFxRajkE
related_formulas: [shannon-entropy]
---

## The unit everything is measured in

Your bill, your context limit, your latency and your rate limits are all
denominated in tokens, so a vague sense of what one is will quietly mislead
you about all four. The usual rule of thumb — about four characters, or
three-quarters of a word, in English prose — holds reasonably well for
ordinary sentences and breaks down exactly where it matters: in code, in
JSON, in long identifiers, in numbers, and in languages other than English.

## Building the vocabulary, one merge at a time

Byte-pair encoding starts from individual characters (strictly, bytes) and
builds up. Take a large corpus, count every pair of adjacent symbols, and
merge the single most frequent pair into one new symbol. Repeat against the
now-slightly-coarser corpus, over and over, until the vocabulary reaches its
target size — typically in the range of 50,000 to 200,000 entries depending
on the model family. A toy illustration makes the mechanics concrete: given a
corpus containing the words "lower", "lowest", "newer" and "wider" many times
over, the pair `e` followed by `r` is extremely common, so it merges first
into a single symbol `er`. Then `low` followed by `er` is common, merging into
`lower` as one token. The word "lowest" does not share that ending, so it
stays split into pieces like `low` and `est`. Nothing here is told what a
word is; the algorithm only ever asks what co-occurs often enough to be worth
spending a vocabulary slot on.

The consequence that matters in practice: a tokenizer's vocabulary is a frozen
snapshot of whatever text it was trained on. Anything that text was rich in —
common English words, common code patterns — gets compressed into few tokens.
Anything it was sparse in gets spelled out letter by letter or close to it.

## The practical weirdness, with real shapes

Leading whitespace is usually fused into the following token, so `"hello"`
and `" hello"` are different token ids — which is why a stray space at the
start of a few-shot example can quietly change how the rest of the completion
is tokenized. Long numbers fragment in ways that do not follow place value:
a four-digit number might become one token, or three, depending on which
specific digit groupings happened to be frequent in the training corpus, which
is a real contributor to why models are unreliable at multi-digit arithmetic
on numbers written out in full. Code suffers in its own way: an indentation
run, an unusual variable name, or a long hexadecimal string can cost far more
tokens than the same number of characters of English prose. And a tokenizer
trained mostly on English text will tokenize non-English scripts far less
efficiently — a sentence that is six tokens in English can easily be three or
four times that in a language the corpus barely covered, which directly
inflates both cost and effective context usage for non-English applications.

## Counting before you ship

Because pricing and context limits are both token-denominated, it is worth
counting tokens programmatically during development rather than guessing from
word counts — the major providers publish the exact tokenizer their models
use as a small library, and running your actual prompts and documents through
it before deployment catches budget surprises long before a user does. A
tokenizer is specific to one model family; counting with one model's
tokenizer and billing against a different model's endpoint will not
reconcile, which matters the moment a system calls more than one model.

This lesson stops at the mechanics. What the budget means once you are
spending it is the
[context window](/roadmaps/ai-engineer/the-context-window).
