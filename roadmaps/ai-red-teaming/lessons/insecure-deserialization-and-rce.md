---
title: Insecure deserialization and RCE in model files
description: Pickle-format model checkpoints can embed executable code, so loading a "model" can secretly mean running an attacker's instructions.
roadmap: ai-red-teaming
node: insecure-deserialization-and-rce
group: system-attacks
video:
  provider: youtube
  id: "IHzn9BiH6rY"
  start: 0
  end: null
creator:
  name: DEFCONConference
  channel_url: https://www.youtube.com/@DEFCONConference
duration_seconds: 1200
level: advanced
prerequisites: [threat-modeling-for-ai-systems]
takeaways:
  - "Python's pickle format can serialize executable instructions, not just data, so deserializing a pickled file can silently run arbitrary code chosen by whoever created it."
  - Model files are pulled from public hubs and loaded directly far more routinely than a normal third-party dependency, which turns pickle-based RCE into a supply-chain problem rather than an isolated bug.
  - Formats like safetensors store only tensor data with no path to code execution, which removes the vulnerability class entirely instead of trying to detect malicious payloads after the fact.
  - A model file behaving correctly on every input you tested tells you nothing about what it did once, silently, the moment it was loaded.
check_questions:
  - q: Why can loading a pickled model file result in code execution rather than just loading a set of weights?
    a: Pickle's protocol lets an object define a method that tells the unpickler to call an arbitrary function with arbitrary arguments as part of reconstructing that object, so a crafted file can make that call run any code its author chose during the act of loading, not only during later execution.
  - q: Why is this treated as an AI-specific supply-chain risk rather than a generic application-security bug?
    a: Pretrained model files are downloaded from public hubs and loaded directly into running processes constantly, usually without the dependency review, hash pinning, or provenance checks a normal third-party library would receive before it ships.
  - q: What is one concrete way to reduce this risk when loading a model you did not train yourself?
    a: Prefer a format such as safetensors that encodes only tensor data with no executable path, and if a pickle-based file is unavoidable, scan it with a dedicated tool and load it inside an isolated, network-restricted environment first.
deeper:
  - label: "DEF CON 30 - Backdooring Pickles: A decade only made things worse — ColdwaterQ"
    url: https://www.youtube.com/watch?v=lECEXFtVjig
related_formulas: []
---

## Loading a model is running a program

Treat "load the model" as a harmless, read-only step and you've already missed the
vulnerability. In most Python machine-learning tooling, a saved model — the
`.pt`, `.pth`, or `.ckpt` file you download from a hub and pass to
`torch.load` — is serialized with Python's built-in `pickle` module. Pickle
was never designed to store only inert data. Its format supports instructing
the unpickler, while reconstructing an object, to call an arbitrary function
with arguments the file's author chose. That mechanism exists so pickle can
faithfully reconstruct complex Python objects — custom classes, closures,
anything with nontrivial `__init__` behavior. It also means a `.pt` file can
contain a call to `os.system`, a reverse shell, or a payload downloader, and
that call fires the instant the file is deserialized, before any inference
ever happens and before the victim has looked at a single prediction. The
DEF CON 33 demonstration this lesson is built around makes that concrete: a
chatbot that looks completely normal in every other respect quietly executes
attacker-chosen code the moment its weights file is loaded, with no error, no
warning, and no observable difference in behavior to the person who loaded
it.

## It isn't only pickle

Pickle is the best-known case, but the same underlying pattern — a format
that is supposed to carry only data ending up with a path to code execution —
recurs across the model-file ecosystem. Older Keras HDF5 checkpoints can
embed a `Lambda` layer that serializes an arbitrary function alongside the
architecture. YAML loaders that default to unsafe parsing can be coerced into
instantiating arbitrary Python objects from a config file, the same class of
bug that hit several real-world YAML-consuming libraries well before anyone
was thinking about AI specifically. Even formats designed to carry only
numeric arrays, like NPZ or certain ONNX custom-operator extensions, have had
implementations found vulnerable to malformed input triggering memory
corruption or arbitrary code paths in the parser itself. The pattern to
internalize isn't "pickle is uniquely bad" — it's that any format expressive
enough to reconstruct rich objects, or any parser complex enough to have
bugs, is a candidate for this class of attack, and model-serving code has
historically trusted these formats far more than the threat model justifies.

## Why this is a supply-chain risk, not a one-off bug

A web application's dependencies usually go through some kind of review: a
package manager, a lockfile, maybe a security scanner in CI. Pretrained model
weights almost never go through anything comparable. A developer finds a
promising checkpoint on a public model hub, pulls it directly into a
training or inference script, and runs it — often on a machine with real
credentials, real network access, and real data sitting nearby. The file
looks like data (a `.bin` or `.pt` extension, a reasonable file size, a
plausible-sounding model card) and is treated like data, even though it is
executable. Multiplied across an ecosystem where reusing someone else's
checkpoint is the norm rather than the exception, this is a genuine
supply-chain vector: compromise or impersonate one popular upload, and every
downstream user who loads it hands you code execution on their machine.

## Loading more safely

The most effective fix is structural rather than procedural: prefer
`safetensors`, a format designed from the start to hold only tensor data with
no mechanism for arbitrary code execution during load, and increasingly the
default output format for major model hubs and training frameworks. Where a
pickle-based file can't be avoided — an older checkpoint, a research
artifact, a format a particular tool still demands — scanning tools exist
that statically inspect a pickle stream for dangerous opcodes before it's
ever deserialized, and loading the file inside a sandboxed, network-isolated
environment limits the blast radius if a scan misses something. None of this
replaces the basic hygiene of only loading files from sources you actually
trust, but trust itself is exactly the thing a convincing-looking public
upload is designed to exploit.
