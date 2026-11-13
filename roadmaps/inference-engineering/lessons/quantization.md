---
title: Quantization
description: Reducing numerical precision to cut memory and increase speed — the quality cost, and post-training quantization versus quantization-aware training.
roadmap: inference-engineering
node: quantization
group: speeding-up-inference
video:
  provider: youtube
  id: "7hYSmzIZXNs"
  start: 0
  end: null
creator:
  name: Cloud Codes
  channel_url: https://www.youtube.com/@Cloud-Codes
duration_seconds: 587
level: advanced
prerequisites: [memory-bound-vs-compute-bound, inference-engines]
takeaways:
  - Quantization reduces a model's weights (and sometimes activations) from a high-precision format like 16-bit floating point to a lower-precision one like 8-bit or 4-bit, directly shrinking memory footprint.
  - Post-training quantization converts an already-trained model's weights after the fact, which is fast and simple but can lose more quality than training the model to tolerate lower precision from the start.
  - Quantization-aware training simulates reduced precision during training itself, so the model learns weights that remain accurate even after quantization, usually at better quality than post-training methods for the same bit width.
check_questions:
  - q: Why does quantization directly help with a memory-bound bottleneck?
    a: Memory-bound operations are limited by how fast data moves, not by how much compute is available. Quantization reduces the number of bytes the weights occupy, so the same memory bandwidth can move more of the model's actual information per second, directly addressing the bottleneck.
  - q: Why does quantization-aware training typically preserve more quality than post-training quantization at the same bit width?
    a: Post-training quantization rounds weights that were never trained with that rounding in mind, so the model has no opportunity to compensate for the resulting error. Quantization-aware training simulates the lower precision during training itself, letting the model's weights adjust to remain accurate despite it.
deeper:
  - label: "Joydeep Bhattacharjee — LLM Quantization Techniques Explained (GPTQ, AWQ, GGUF, HQQ, BitNet)"
    url: https://www.youtube.com/watch?v=0pF6GdbwMo4
related_formulas: [model-memory-footprint]
---

## Fewer bits, same information, less memory

A model's weights are ordinarily stored as 16-bit or 32-bit floating-point
numbers. **Quantization** converts them to a lower-precision format — 8-bit,
4-bit, or lower — storing an approximation of each original value using
fewer bits. This directly shrinks how much memory the weights occupy, per
[model memory footprint](/formulas/model-memory-footprint), and, since
decode is typically memory-bandwidth-bound as covered in
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound), it directly speeds up exactly the phase that
bottleneck affects — the same memory bandwidth now has to move fewer bytes
to deliver the same number of parameters.

## The quality cost, and why it is usually worth paying

Approximating each weight with fewer bits necessarily loses some precision,
and that loss shows up as some degradation in the model's output quality.
The practical finding across this field is that this cost is frequently
small relative to the memory and speed gained — a model quantized to 8-bit,
and often even 4-bit, performs close enough to its full-precision original
on most tasks that the tradeoff is clearly worth it, though how much quality
is lost varies by model, by task, and by exactly how the quantization is
done.

## Two ways to get there

**Post-training quantization (PTQ)** takes an already-fully-trained model and
converts its weights to lower precision afterward, as a separate step with no
further training involved. This is fast and simple — it can be applied to
any existing checkpoint in minutes — but because the model was never trained
with this rounding in mind, it has no opportunity to compensate for the
resulting approximation error, which can mean a larger quality hit,
especially at very low bit widths.

**Quantization-aware training (QAT)** instead simulates the effect of
reduced precision during training itself, so the model's weights are
optimised with the eventual rounding already accounted for. This typically
preserves more quality at the same target bit width than PTQ achieves, at the
cost of requiring an actual training process rather than a quick post-hoc
conversion — meaningfully more expensive to produce, but often worth it when
quality at low precision genuinely matters.

## Where quality loss actually shows up

Quality degradation from quantization is rarely uniform across a model's
capabilities. A quantized model often performs nearly indistinguishably from
its full-precision original on straightforward tasks, while showing a more
noticeable drop on tasks demanding precise numerical reasoning or very long,
coherent generation — the kinds of tasks where small, compounding
imprecisions in individual weight values have more opportunity to accumulate
into a visibly different final answer. This is why evaluating a quantized
model deserves the same discipline covered in Upshard's AI Engineer roadmap's
treatment of evals: a quick qualitative glance at a few outputs can look fine
while a real eval set, specifically including the harder tasks a product
actually depends on, reveals a degradation a casual check would have missed
entirely.

## The named methods worth recognising

**GPTQ** and **AWQ** are both popular post-training quantization methods
that go beyond naive rounding — using more sophisticated techniques
(Hessian-based reconstruction for GPTQ, activation-aware scaling for AWQ) to
minimise the quality loss PTQ would otherwise incur. **GGUF** is primarily a
file format associated with the `llama.cpp` ecosystem, commonly paired with
quantized weights for local and edge deployment, covered in
[local and edge inference](/roadmaps/inference-engineering/local-and-edge-inference). Recognising these names is useful mainly because they appear
constantly in practice — knowing that GPTQ and AWQ are both PTQ methods with
different internal tricks, rather than fundamentally different approaches, is
usually enough to choose between them for a real deployment.
