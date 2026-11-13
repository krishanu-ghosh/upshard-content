---
title: Embedding and multimodal inference
description: Serving embedding models and vision-language models — different bottlenecks than text generation, since there is often no multi-step decode at all.
roadmap: inference-engineering
node: embedding-and-multimodal-inference
group: scaling-and-beyond-llms
video:
  provider: youtube
  id: "PcvxdWJOyUE"
  start: 0
  end: null
creator:
  name: Nadav Timor
  channel_url: https://www.youtube.com/@NadavTimor
duration_seconds: 3660
level: advanced
prerequisites: [attention-variants-for-serving]
takeaways:
  - An embedding model produces its entire output in a single forward pass with no decode phase at all, which makes it closer in performance profile to prefill alone than to the full generate-one-token-at-a-time workload the rest of this roadmap assumes.
  - A vision-language model adds an image or video encoding step before the usual text pipeline, which introduces its own compute cost and its own token budget before a single word of text has been processed.
  - Serving infrastructure built around text generation's prefill-then-decode shape does not automatically fit these workloads well, and treating them as identical to text generation is a common source of poorly-tuned deployments.
check_questions:
  - q: Why is serving an embedding model a fundamentally different workload from serving a text-generation model?
    a: An embedding model produces its complete output — a fixed-size vector — in one forward pass, with nothing resembling the sequential, memory-bound decode phase that dominates text generation. Its performance profile looks much more like prefill alone, which is compute-bound, than like the full generate-many-tokens workload this roadmap mostly assumes.
  - q: What extra cost does a vision-language model introduce before any text processing even begins?
    a: An image or video first has to be encoded into a form the language model component can process, which is its own compute step with its own cost — commonly translating into a large number of effective tokens an image contributes before any of the actual text in the prompt has been read at all.
deeper: []
related_formulas: []
---

## A genuinely different shape of workload

Every lesson in this roadmap so far has assumed the shape of autoregressive
text generation: a prefill pass over the prompt, followed by a sequential
decode loop producing one token after another. Embedding models and
vision-language models do not share this shape, and serving them well means
recognising exactly how they differ rather than assuming the same techniques
apply unchanged.

## Embedding models: prefill, and nothing else

An embedding model takes a piece of text (or an image, for some models) and
produces a single fixed-size vector in one forward pass — there is no
decode phase at all, because there is nothing to generate token by token; the
entire output already exists once that one pass completes. This makes an
embedding model's performance profile much closer to prefill alone, as
covered in
[prefill and decode](/roadmaps/inference-engineering/prefill-and-decode) —
largely compute-bound, highly parallelisable, with none of the sequential,
memory-bandwidth-bound decode loop that dominates the cost of serving a
text-generation model. Batching many embedding requests together is
correspondingly simpler and more effective than batching generation
requests, precisely because there is no variable-length decode loop to
coordinate across the batch.

## Why embedding batching is simpler to get right

A text-generation batch has to track, for every request in it, how many
tokens it has generated so far and whether it has finished — state that
changes independently for each request every single step. An embedding batch
has none of that: every request in it runs for exactly one forward pass and
produces exactly one output, so there is no variable-length bookkeeping at
all. This is why an embedding-serving system can often get away with a much
simpler scheduler than a text-generation engine needs, and why pushing
embedding traffic through infrastructure built for the harder problem is
usually unnecessary overhead rather than a safety margin.

## Vision-language models: an encoding step before any text

A **vision-language model (VLM)** processes an image or video alongside text,
and before any of the text pipeline runs at all, the visual input has to be
encoded into a form the language model component can actually consume — a
separate compute step with its own real cost. In practice this commonly
means an image contributes a large number of effective tokens to the
prompt's budget, sometimes more than a substantial amount of accompanying
text would, which has to be accounted for in exactly the way a text request's context budget is
tracked, as covered in
[the inference stack](/roadmaps/inference-engineering/the-inference-stack). A video adds a further dimension:
frames sampled over time each carry this same encoding cost, so the sampling
rate chosen becomes a direct and significant cost and latency lever, in
exactly the way covered for multimodal inputs in Upshard's AI Engineer
roadmap.

## Why generic text-serving infrastructure does not automatically fit

An inference engine tuned purely around the assumption of a prefill-then-
decode text workload can handle embedding and multimodal requests, but not
necessarily well: an embedding workload gains little from decode-oriented
optimisations like
[speculative decoding](/roadmaps/inference-engineering/speculative-decoding), since there is no decode loop to speed up at all, and a
vision-language workload's image-encoding step is a cost category the usual
prefill/decode accounting does not capture on its own. Recognising that
these are genuinely different workload shapes, rather than minor variations
on text generation, is the first step toward provisioning and tuning serving
infrastructure that actually fits them.
