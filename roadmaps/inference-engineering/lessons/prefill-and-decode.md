---
title: Prefill and decode
description: The two phases of generating a response — prefill processes the prompt in parallel, decode generates tokens one at a time — with opposite performance profiles.
roadmap: inference-engineering
node: prefill-and-decode
group: inside-the-forward-pass
video:
  provider: youtube
  id: "HRKFa8LIAQg"
  start: 0
  end: null
creator:
  name: Ready Tensor
  channel_url: https://www.youtube.com/@ready_tensor
duration_seconds: 872
level: advanced
prerequisites: [the-inference-stack, time-to-first-token]
takeaways:
  - Prefill processes the entire input prompt in a single parallel pass, computing attention over every input token at once — it is compute-bound, and the GPU's full throughput is put to use.
  - Decode generates output tokens one at a time, each requiring its own full pass through the model conditioned on everything generated so far — it is sequential, and typically memory-bandwidth-bound rather than compute-bound.
  - Every request passes through exactly one prefill phase followed by a decode phase of however many tokens are generated, and the two phases have such different resource profiles that serving systems increasingly handle them differently.
check_questions:
  - q: Why can prefill be done in a single parallel pass while decode cannot?
    a: Every position in the input prompt is already known at the start, so the model can compute attention and activations for every position simultaneously. Each decode step produces a new token that the next step depends on, so decode steps cannot happen out of order or in parallel with each other.
  - q: Why does the GPU's compute capacity sit mostly idle during decode?
    a: Each decode step processes only one new token per request, which is too little arithmetic work to keep the GPU's compute units busy. The bottleneck instead becomes how fast the GPU can read the model's weights from memory for that one token, which is why decode is usually memory-bandwidth-bound.
deeper:
  - label: "Databricks — Accelerating LLM Inference with vLLM (Zhuohan Li, Cade Daniel)"
    url: https://www.youtube.com/watch?v=qBFENFjKE-M
related_formulas: [transformer-forward-flops, memory-bound-token-latency]
---

## One request, two completely different phases

Generating a response to a prompt happens in two phases with almost opposite
performance characteristics. **Prefill** processes every token of the input
prompt at once, in a single parallel pass — computing every position's
attention and activations simultaneously, because the entire prompt is
already known before generation starts. **Decode** then produces the output
one token at a time, and each new token has to wait for the previous one,
because it is conditioned on everything generated so far, including the
token decode just produced.

## Why prefill is compute-bound

During prefill, the GPU processes the whole prompt at once, which means a
genuinely large amount of matrix multiplication happening in parallel — see
[transformer forward FLOPs](/formulas/transformer-forward-flops) for how that
scales with prompt length and model size. This keeps the GPU's compute units
close to fully busy, which is exactly the regime a GPU is built to excel at.
Prefill is, in the terms covered in
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound), usually compute-bound — the arithmetic, not memory
movement, is the limiting factor.

## Why decode is memory-bound

Each decode step produces exactly one new token, which means comparatively
little arithmetic work per step. What dominates instead is reading the
model's entire set of weights from GPU memory for that one token's
computation — see
[memory-bound token latency](/formulas/memory-bound-token-latency) for the
arithmetic behind this. Because so little compute happens per step relative
to how much data has to move, decode is typically memory-bandwidth-bound: the
GPU's compute units sit comparatively idle while the system waits on memory
bandwidth, the opposite regime from prefill.

## A concrete walk through one request

Trace a single request end to end to see both phases in sequence. A user
sends a five-hundred-token prompt. Prefill processes all five hundred tokens
in one parallel pass, computing attention across the whole prompt at once —
this is the bulk of what [time to first token](/roadmaps/inference-engineering/time-to-first-token) measures. The moment prefill finishes, decode
begins: token one is generated, then fed back in as additional context for
token two, which is generated and fed back in for token three, and so on,
one sequential step at a time, until the response is complete or a stopping
condition is reached. If the response runs to two hundred tokens, that is
two hundred separate sequential decode steps, each one a full pass through
the model's weights for a single new token — which is why a long response
costs so much more than a long prompt of the same token count, even though
both get processed by the identical model.

## Why the split shapes the rest of this roadmap

This single distinction — one compute-bound phase, one memory-bound phase,
per request — is the reason behind much of what the rest of this roadmap
covers. It is why
[continuous batching](/roadmaps/inference-engineering/continuous-batching)
helps decode specifically, by letting many requests' decode steps share a
single GPU pass. It is why
[disaggregated serving](/roadmaps/inference-engineering/disaggregated-serving) exists at all — since the two phases have such different resource
profiles, separating them onto differently-provisioned hardware pools can use
both more efficiently than forcing one machine to serve both. And it is the
underlying reason [KV caching](/roadmaps/inference-engineering/kv-cache-mechanics), covered in the next lesson, exists in the first place: without
it, every single decode step would have to redo the attention computation
over the entire prompt again, turning a cheap sequential step into a
repeated, expensive one.
