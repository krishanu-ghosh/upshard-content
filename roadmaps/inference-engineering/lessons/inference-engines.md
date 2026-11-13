---
title: Inference engines
description: What a dedicated inference engine like vLLM, TensorRT-LLM or SGLang does beyond running the model forward — and why it beats calling a raw framework directly.
roadmap: inference-engineering
node: inference-engines
group: serving-software
video:
  provider: youtube
  id: "Ahtaha9fEM0"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 2622
level: advanced
prerequisites: [gpu-architecture-for-inference, prefill-and-decode]
takeaways:
  - A dedicated inference engine handles continuous batching, KV cache memory management and optimized kernels automatically — work a raw model framework call does not do for you at all.
  - vLLM, TensorRT-LLM and SGLang are the three engines most commonly reached for in production, each with different strengths, and the gap between any of them and a naive implementation is typically far larger than the gap between the engines themselves.
  - A model is distributed as a checkpoint in some format — safetensors is the common default on the Hugging Face ecosystem, ONNX as a portable interchange format, framework-native formats elsewhere — and converting between them is a real step with real failure modes, not a trivial file copy.
check_questions:
  - q: What does a dedicated inference engine do that calling a model framework directly does not?
    a: It automatically batches incoming requests together efficiently, manages the KV cache's memory across many concurrent requests without waste, and runs the model's forward pass through kernels specifically optimized for serving rather than for training.
  - q: Why might converting a model checkpoint from one format to another go wrong?
    a: A format encodes more than raw numbers — architecture metadata, tokenizer configuration, sometimes executable code paths. A naive or incomplete conversion can silently drop or misinterpret some of this, producing a model that loads without error but behaves subtly differently from the original.
deeper:
  - label: "Nadav Timor — Accelerating LLM Inference with vLLM and SGLang (Ion Stoica)"
    url: https://www.youtube.com/watch?v=PcvxdWJOyUE
  - label: "NVIDIA Developer — Beyond the Algorithm: The New PyTorch Architecture for TensorRT-LLM"
    url: https://www.youtube.com/watch?v=M5wSNxZvbJ8
related_formulas: []
---

## Everything a naive forward pass does not do for you

Calling a model framework directly — load the weights, run a forward pass,
read the output — technically produces inference, but it does none of the
work that makes serving real traffic efficient. It processes one request at
a time with no batching. It recomputes or wastes memory on the KV cache with
no particular care. It runs through whatever kernels the framework ships
with, which were generally written with training, not serving, in mind. A
dedicated **inference engine** exists specifically to do all of this well:
[vLLM](https://github.com/vllm-project/vllm), **TensorRT-LLM**, and
**SGLang** are the three most widely used in production today, and each
automates continuous batching, KV cache memory management, and
serving-optimized kernel execution as its core job.

## Why the gap between engines matters less than it sounds

These three engines differ in real ways — their scheduling policies, which
hardware vendors they are most optimized for, specific advanced features one
has that another lacks — and reasonable engineering teams choose between
them based on real tradeoffs covered partly in
[benchmarking inference systems](/roadmaps/inference-engineering/benchmarking-inference-systems). But the gap between any of them and a naive,
unoptimized implementation is typically far larger than the gap between the
engines themselves. The first and most important decision is using a
dedicated engine at all; which specific one is a real but secondary question.

## Formats: what actually moves between systems

A trained model is distributed as a **checkpoint** in one of several formats.
**Safetensors** has become the default on the Hugging Face ecosystem
specifically because it stores only raw tensor weights with no embedded
executable code, which makes it fast to load and safe to download from
untrusted sources — unlike Python's older pickle format, which can execute
arbitrary code on load. **ONNX** is a portable interchange format that
captures not just weights but the model's full computation graph, intended
to let a model trained in one framework run in a different runtime
altogether. Framework-native formats (a raw PyTorch checkpoint, for instance)
are the simplest to produce but the least portable across tooling.

Converting between these is a real engineering step, not a trivial file
copy: a format encodes architecture metadata and tokenizer configuration
alongside the raw numbers, and an incomplete or careless conversion can
silently drop or misinterpret some of it, producing a model that loads
without any error at all and still behaves subtly differently from the
original. Whichever inference engine you choose typically has a preferred
format, and picking a model distribution that already matches it avoids an
entire category of conversion bugs before they happen.

A quieter version of the same risk shows up even without a format
conversion: two checkpoints of the "same" model, downloaded from different
sources, can disagree on tokenizer configuration or a default generation
setting while both loading without error. The failure mode is identical —
a model that behaves subtly differently from what you expected, discovered
only once it is already serving traffic — which is why pinning an exact
checkpoint source and verifying outputs against a known reference, rather
than trusting that "the same model name" means byte-identical behavior, is
worth the extra step before a new checkpoint goes anywhere near production.
