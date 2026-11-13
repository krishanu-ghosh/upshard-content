---
title: Activation functions
description: Sigmoid, tanh and ReLU — what each does to its input, the vanishing gradient problem, and why ReLU became the default for hidden layers.
roadmap: machine-learning
node: activation-functions
group: neural-network-foundations
video:
  provider: youtube
  id: "68BZ5f7P94E"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 538
level: intermediate
prerequisites: [neural-network-basics]
takeaways:
  - An activation function is what lets stacked layers represent more than a single linear transformation — without one between every layer, a deep network collapses mathematically into a shallow linear model.
  - Sigmoid and tanh squash their input into a bounded range, but their gradient becomes extremely small for large inputs in either direction, which makes a deep network built from them very slow or unable to learn in its earliest layers.
  - ReLU simply passes positive inputs through unchanged and outputs zero for negative inputs; its gradient never shrinks for positive inputs, which is why it became the default choice for hidden layers in most modern networks.
check_questions:
  - q: Why does a network built entirely from sigmoid activations struggle to train when it has many layers?
    a: Sigmoid's gradient is near zero for large positive or negative inputs. During training, the error signal gets multiplied by this small gradient at every layer it passes backward through, shrinking rapidly as it propagates — by the earliest layers of a deep network, the signal can become too small to produce any meaningful weight update at all.
  - q: What makes ReLU's gradient behaviour different from sigmoid's for positive inputs?
    a: ReLU's output is simply equal to its input whenever that input is positive, so its gradient for any positive input is exactly one — constant, and never shrinking no matter how large the input gets, unlike sigmoid's gradient which shrinks toward zero as its input grows large in either direction.
deeper:
  - label: "StatQuest — The GELU, SiLU and SwiGLU activation functions"
    url: https://www.youtube.com/watch?v=2FaI2Fen1mQ
related_formulas: [sigmoid, relu, gelu]
---

## The step that makes depth worth anything

As the previous lesson established, stacking purely linear layers collapses
mathematically into a single linear transformation, no matter how many layers
are stacked. An **activation function** applied after every layer's weighted
sum is what prevents this collapse — it is a non-linear function, and
inserting one between every pair of layers is precisely what lets a deep
network represent relationships a shallow linear model structurally cannot.

## Sigmoid and tanh: bounded, and a real problem hiding inside

[Sigmoid](/formulas/sigmoid), already introduced for turning a linear output
into a probability in
[logistic regression](/roadmaps/machine-learning/logistic-regression),
squashes any real number into the range between zero and one. **Tanh**
performs a similar squashing into the range between negative one and one.
Both were the standard choice for hidden-layer activations in early neural
networks, and both share a serious problem once networks get deep: their
gradient — how much the output changes for a small change in input — becomes
extremely small whenever the input is large in either direction, in a region
often called the saturated part of the curve.

## Why a small gradient becomes a training disaster at depth

Training a neural network works backward from the output, propagating an
error signal through every layer in reverse, covered properly in the next
lesson on backpropagation. At each layer that signal gets multiplied by that
layer's local gradient. If every layer uses sigmoid or tanh, and several of
those layers' inputs happen to sit in the saturated region, the error signal
gets multiplied by a series of very small numbers in succession, shrinking
rapidly as it travels backward. By the time that shrunk signal reaches the
earliest layers of a genuinely deep network, it can become too small to
produce any meaningful update to those layers' weights at all — this is the
**vanishing gradient problem**, and it made training deep networks with these
activations notoriously difficult for years.

## ReLU: simple, and it does not saturate on the positive side

**ReLU** (rectified linear unit) is almost disarmingly simple: it outputs its
input unchanged whenever that input is positive, and outputs exactly zero
whenever it is negative. Its gradient for any positive input is a constant
one — it never shrinks, no matter how large the input grows, which directly
solves the vanishing-gradient problem on the positive side. This simplicity,
combined with being extremely cheap to compute, is why ReLU became the
default choice for hidden layers across most modern network architectures,
displacing sigmoid and tanh almost entirely from that specific role.

ReLU is not without its own issue — a neuron whose weights happen to push its
input permanently negative outputs zero forever afterward and stops
contributing any gradient at all, sometimes called a "dead" neuron. Variants
such as **leaky ReLU**, which allows a small non-zero slope for negative
inputs instead of a hard zero, and newer activations such as **GELU**, used
in many current large models and covered in the deeper link, address this and
other refinements while keeping ReLU's core advantage over sigmoid and tanh
largely intact.

## Where each kind still earns a place

Sigmoid has not disappeared entirely — it remains the right choice at an
output layer specifically producing a probability, exactly as in
[logistic regression](/roadmaps/machine-learning/logistic-regression),
precisely because a probability genuinely needs to be bounded between zero
and one. The lesson here is narrower than "always use ReLU": it is that the
choice of activation function depends on where in the network it sits and
what that layer is actually meant to produce, with ReLU and its variants as
the strong default for hidden layers specifically.
