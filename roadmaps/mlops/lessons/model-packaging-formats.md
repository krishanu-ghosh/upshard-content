---
title: Model packaging formats
description: Portable serialization formats like ONNX let you train in one framework and serve with a different, often faster, runtime.
roadmap: mlops
node: model-packaging-formats
group: packaging
video:
  provider: youtube
  id: "0t2jOBZSd6s"
  start: 0
  end: null
creator:
  name: DigitalSreeni
  channel_url: https://www.youtube.com/@DigitalSreeni
duration_seconds: 865
level: intermediate
prerequisites: [containerizing-models]
takeaways:
  - A framework-native checkpoint (a PyTorch .pt file, a TensorFlow SavedModel) only runs inside the framework that produced it, which couples your serving stack to your training stack even when that coupling buys you nothing at inference time.
  - A portable format like ONNX represents a model as a framework-independent computation graph, so a model trained in one framework can be loaded and executed by a completely different runtime.
  - Exporting to a portable format is also an opportunity for graph-level optimizations (operator fusion, constant folding) that a dedicated inference runtime applies automatically and a general-purpose training framework does not.
check_questions:
  - q: Why can't a model saved in PyTorch's native checkpoint format be loaded directly by a TensorFlow serving process?
    a: Because a native checkpoint stores framework-specific weight tensors and often framework-specific code references, with no framework-independent description of the model's computation graph that another framework's runtime could interpret.
  - q: What does an ONNX file actually contain?
    a: A framework-independent graph of standardized mathematical operators along with the trained weight values, so any runtime that implements the ONNX operator set can execute it without needing the original training framework installed.
  - q: Besides portability, what's a second concrete benefit of exporting a model to a format like ONNX before serving it?
    a: The export step lets a dedicated inference runtime apply graph-level optimizations, such as fusing multiple operators into one or folding constant computations ahead of time, that typically make inference measurably faster than running the same model inside its original training framework.
deeper:
  - label: "Converting Models to #ONNX Format — ONNX Runtime"
    url: "https://www.youtube.com/watch?v=lRBsmnBE9ZA"
related_formulas: []
---

## The coupling nobody asked for

Train a model in PyTorch and the natural thing to do is save it with
`torch.save` and load it later with `torch.load`. That works fine right up
until the moment your serving requirements diverge from your training
requirements — which they almost always eventually do. Maybe the team that
owns production inference standardizes on a C++ runtime for latency
reasons. Maybe you want to deploy the same model to a mobile device, where
shipping the full PyTorch runtime is impractical. Maybe a different team
needs to run the model inside a Java service. A native checkpoint ties you
to the framework that produced it, not because the *model* requires that
framework to run — mathematically, a trained model is just a graph of
operations over tensors — but because the checkpoint format only encodes
enough information for that one framework's runtime to reconstruct it.

This is a coupling with no inference-time benefit. Training frameworks
like PyTorch and TensorFlow are built for an entirely different job than
serving: they need to support automatic differentiation, dynamic graph
construction, and a huge general-purpose operator surface, because
researchers need that flexibility while building and debugging models.
None of that flexibility is needed once a model is trained and frozen for
inference — at that point you have a fixed, static computation graph, and
the only things that matter are "run it correctly" and "run it fast."
Portable model formats exist to let you cross that boundary deliberately.

## What a portable format actually is

ONNX (Open Neural Network Exchange) is the clearest example: it defines a
standardized set of mathematical operators (convolution, matrix multiply,
batch normalization, and so on) and a graph format for connecting them,
independent of any specific framework's implementation. Exporting a model
to ONNX means walking the trained model's computation graph and
translating each framework-specific operation into the equivalent
standardized ONNX operator, then saving that graph together with the
trained weight tensors into a single `.onnx` file. Nothing about that file
references PyTorch or TensorFlow internals anymore — it's a
self-describing computation graph plus data.

Any runtime that implements the ONNX operator specification can then load
and execute that graph, regardless of what trained it. ONNX Runtime is the
reference implementation, but the same file can also run inside
TensorRT (NVIDIA's inference optimizer), on mobile through ONNX Runtime
Mobile, or inside a browser via ONNX Runtime Web. You trained once, in
whatever framework your research team is most productive in, and you can
now serve that exact model from whichever runtime best fits each
deployment target, without re-implementing or retraining anything. The
export is a translation, not a retraining — the numerical behavior of the
model is preserved (modulo small floating-point differences from
operator-level implementation details), so the thing you validated during
training is the thing that ships.

## Speed isn't incidental, it's structural

Portability is the headline reason to use a format like ONNX, but the
performance gain that comes with it is not a side effect — it follows
directly from the fact that a frozen inference graph can be optimized in
ways a dynamic training graph cannot. A dedicated inference runtime looks
at the static ONNX graph and can fuse sequences of operators that always
run together (for instance, a convolution immediately followed by a batch
normalization and a ReLU) into a single kernel call, eliminating the
overhead of launching several separate operations and writing intermediate
results to memory between them. It can fold any sub-computation whose
inputs are already known at export time — constants baked in during
training — directly into fixed values, so that work never happens again at
inference. It can also apply hardware-specific kernel selection, picking
the fastest available implementation of each operator for the exact GPU or
CPU the model will actually run on. A training framework generally skips
these optimizations because they'd make the graph harder to modify
mid-training — exactly the flexibility training needs and inference
doesn't.

## Portability as the default, not the exception

The practical takeaway is to treat framework-native checkpoints as a
training-time artifact, not a deployment artifact. Once a model is
validated and ready to ship, export it to a portable format as a
deliberate packaging step — the same way the previous lesson treated
pinning dependencies as a deliberate step rather than an afterthought.
Doing this decouples your serving infrastructure's technology choices from
your research team's framework preferences, and it gives you the option to
move a model to a faster or more suitable runtime later without touching
the training code at all. Combined with containerization, a portable model
format is what makes "train anywhere, serve anywhere" an actual operating
model instead of an aspiration — which is exactly the foundation the next
group of lessons, on CI/CD for ML, builds its automation on top of.
