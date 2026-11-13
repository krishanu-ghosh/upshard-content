---
title: Recurrent neural networks
description: Maintaining a hidden state updated at each sequence step, the vanishing gradient problem over long sequences, and LSTM/GRU as the fix.
roadmap: machine-learning
node: recurrent-neural-networks
group: deep-learning-frontiers
video:
  provider: youtube
  id: "YCzL96nL7j0"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 1245
level: advanced
prerequisites: [convolutional-neural-networks]
takeaways:
  - A recurrent network processes a sequence one element at a time, maintaining a hidden state that carries information forward from everything seen so far, rather than seeing the whole sequence at once the way earlier architectures in this roadmap do.
  - Plain RNNs suffer from vanishing gradients over long sequences for the same underlying reason a deep network with saturating activations does — the error signal shrinks as it is propagated backward across many time steps.
  - LSTM and GRU add learned gates that control what information to keep, forget, or output at each step, which lets gradients flow backward across many more time steps without vanishing the way a plain RNN's gradient does.
check_questions:
  - q: What does a recurrent network's hidden state actually carry forward between time steps?
    a: A compressed summary of everything the network has processed in the sequence so far, updated at every step by combining the previous hidden state with the current input — it is how the network has any memory of earlier parts of the sequence while processing a later one.
  - q: Why does an LSTM handle long sequences better than a plain RNN?
    a: Its learned gates explicitly control what to keep, forget, and output at each step, creating a more direct path for gradients to flow backward across many time steps without shrinking the way they do when forced through a plain RNN's repeated, unguarded transformation at every step.
deeper:
  - label: "StatQuest — Recurrent Neural Networks, Clearly Explained"
    url: https://www.youtube.com/watch?v=AsNTP8Kwu80
  - label: "The AI Hacker — Illustrated Guide to LSTM's and GRU's"
    url: https://www.youtube.com/watch?v=8HyCNIVRbSU
related_formulas: []
---

## Data that has an order, not just a position

Every architecture earlier in this roadmap processes its input as a fixed,
complete whole — a row of features, or a whole image at once. Some data is
fundamentally different: a sentence, a time series of sensor readings, a
sequence of user actions, where the order genuinely matters and the sequence
can be of varying, unpredictable length. A **recurrent neural network (RNN)**
is built specifically for this kind of data, processing it one element at a
time while carrying information forward from everything seen so far.

## The hidden state: memory carried forward

At each step, an RNN takes the current input together with a **hidden
state** — a vector summarising everything processed up to that point — and
produces an updated hidden state, which then carries forward into the next
step. This is the mechanism that gives the network any notion of context: by
the time it processes the fifth word of a sentence, its hidden state
theoretically carries some compressed trace of the first four. The same
small set of weights is reused at every single time step, regardless of how
long the sequence is, which is what lets an RNN handle sequences of varying
length without needing a different architecture for each possible length.

## Why plain RNNs struggle with long sequences

Training an RNN means backpropagating error backward through time, across
every step of the sequence, in a process directly analogous to backpropagating through the layers of a deep feedforward network, covered in
[backpropagation](/roadmaps/machine-learning/backpropagation). Over a long
enough sequence, this is mechanically equivalent to a very deep network, and
it suffers the same vanishing gradient problem covered in
[activation functions](/roadmaps/machine-learning/activation-functions): the
error signal shrinks as it propagates backward across many time steps, which
means a plain RNN struggles to learn dependencies between elements that are
far apart in a sequence — a word at the very start of a long passage has
essentially no influence on predictions made many steps later, not because
the information is irrelevant, but because the gradient carrying that
influence backward has vanished by the time it gets there.

## LSTM and GRU: gates that protect the gradient

**LSTM** (long short-term memory) networks address the vanishing gradient
problem directly by adding learned **gates** at each step: a forget gate
decides what to discard from the carried-forward state, an input gate decides
what new information to add, and an output gate decides what to actually
expose as the current output. These gates create a more direct path for
gradients to flow backward across many time steps without the same
shrinkage a plain RNN suffers, which lets an LSTM learn dependencies spanning
much longer sequences. **GRU** (gated recurrent unit) is a simpler variant
using fewer, more combined gates, generally achieving comparable performance
to LSTM with somewhat less computation.

## Where attention eventually replaced this approach

RNNs, and LSTMs especially, were the dominant approach to sequence modelling
for years. The hidden state carried between steps is, however, a bottleneck
— the entire history of a long sequence has to be compressed into one
fixed-size vector, no matter how much information that sequence actually
contains. The next lesson in this roadmap,
[attention and transformers](/roadmaps/machine-learning/attention-and-transformers), covers the architectural idea that eventually replaced this
hidden-state bottleneck for most large-scale sequence modelling — letting a
model look directly at any earlier part of a sequence rather than relying on
a single compressed summary carried step by step.
