---
title: Distributed training infrastructure
description: Why training sometimes needs more than one GPU, and what data and model parallelism actually split across machines to make that possible.
roadmap: mlops
node: distributed-training-infrastructure
group: orchestration-infra
video:
  provider: youtube
  id: "a6_pY9WwqdQ"
  start: 0
  end: null
creator:
  name: Lightning AI
  channel_url: https://www.youtube.com/@PyTorchLightning
duration_seconds: 335
level: advanced
prerequisites: [cloud-ml-platforms]
takeaways:
  - Distributed training becomes necessary when either the dataset makes single-GPU training too slow or the model itself no longer fits in one GPU's memory, and those two problems call for different solutions.
  - Data parallelism replicates the full model on every GPU and splits the data, while model parallelism splits the model itself across GPUs when it's too large to replicate — they solve different bottlenecks and are often combined.
  - The infrastructure underneath distributed training (high-bandwidth interconnects, gradient synchronization, orchestration to place and restart workers) is what makes parallelism actually fast rather than merely possible.
check_questions:
  - q: What are the two distinct reasons a team would need distributed training, and why do they call for different approaches?
    a: A dataset too large to train on in reasonable time calls for data parallelism (splitting data across replicated models), while a model too large to fit in one GPU's memory calls for model parallelism (splitting the model itself), and conflating the two leads to picking the wrong strategy.
  - q: In data parallelism, what has to happen after each GPU computes gradients on its data shard, before the next training step?
    a: The gradients computed on each GPU have to be synchronized (typically averaged) across all GPUs so every replica of the model stays identical, which requires fast communication between GPUs and is often the actual bottleneck.
  - q: Why does distributed training depend so heavily on networking infrastructure specifically, more than single-GPU training does?
    a: Because GPUs must repeatedly exchange gradients or activations during training, and if the interconnect between them is slow relative to each GPU's compute speed, the GPUs spend more time waiting to communicate than they spend computing, which erases the benefit of adding more GPUs.
deeper: []
related_formulas: []
---

## Two different reasons to go distributed

"Distributed training" gets used as one phrase for two genuinely different
problems. The first is a speed problem: your dataset is large enough that
training on one GPU would take weeks, and you want to cut that down by
spreading the work across many GPUs running in parallel. The second is a
capacity problem: your model's parameters, activations, and optimizer state
no longer fit in one GPU's memory at all, regardless of how much time
you're willing to spend — training simply cannot start on a single device.
These call for different solutions, and a lot of confusion in this space
comes from treating "distributed training" as one technique instead of
recognizing which problem you actually have.

## Data parallelism: same model, different data

Data parallelism is the answer to the speed problem. The full model is
replicated identically on every GPU; each replica trains on a different
shard of the data in each step, computing its own forward pass and its own
gradients. Those gradients are then synchronized across all GPUs — usually
averaged — so every replica applies the same update and stays identical
before the next step begins. This is conceptually simple and scales well
when the model fits comfortably on one device: doubling the number of GPUs
roughly doubles the data throughput, in the ideal case. The catch is that
synchronization step. Averaging gradients across, say, 64 GPUs requires
real communication, and if that communication is slow relative to how fast
each GPU finishes its local computation, the GPUs sit idle waiting for each
other instead of training. This is why data-parallel training lives or dies
on the quality of the interconnect between GPUs, not just on how many GPUs
you add.

## Model parallelism: splitting the model itself

Model parallelism is the answer to the capacity problem. When a model is
too large to replicate on a single GPU — common with large language models
whose parameters alone can exceed one device's memory — the model is split
across multiple GPUs instead, with each one holding a different subset of
layers (pipeline parallelism) or a different slice of each layer's weights
(tensor parallelism). A forward pass now has to physically cross GPU
boundaries: GPU 1 computes its layers and sends activations to GPU 2 to
continue, and gradients flow back the same path during backpropagation.
This solves the memory problem but introduces its own bottleneck — GPUs
further along the pipeline sit idle waiting for earlier ones to finish,
unless the work is carefully pipelined so multiple batches are in flight at
once. In practice, large-scale training often combines both: model
parallelism to fit an oversized model across a group of GPUs, and data
parallelism across multiple such groups to get throughput on top of that.

## The infrastructure that makes this actually work

Neither form of parallelism is free, and both depend on infrastructure
most single-GPU training never has to think about. High-bandwidth, low-latency
interconnects between GPUs — technologies like NVLink within a machine and
InfiniBand or specialized networking fabrics between machines — exist
because ordinary Ethernet is far too slow for the volume of gradient and
activation traffic distributed training generates. An orchestration layer
is needed to place workers correctly (which GPU gets which model shard or
data shard), to restart a failed worker without corrupting the whole
training job's state, and to checkpoint progress so a multi-day run
survives a single node failure instead of losing everything. None of this
is optional extra polish — without it, distributed training is either too
slow to be worth the extra hardware or too fragile to complete a long
training run at all.

## Why this matters going forward

Distributed training is where the infrastructure decisions earlier in this
group stop being abstract. Whether you provisioned your own cluster with
infrastructure as code or you're using a managed platform's distributed
training support, the actual performance you get depends on understanding
which bottleneck you're solving — data volume or model size — and whether
the interconnect and orchestration underneath can keep GPUs fed with work
instead of waiting on each other. Getting this wrong doesn't just waste
money on idle GPUs; past a certain scale, it's the difference between a
training run that finishes and one that never converges at all.
