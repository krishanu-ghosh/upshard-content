---
title: Generative models
description: Models that generate new data rather than just predicting a label — autoencoders, learning a compressed representation, and GANs' adversarial setup.
roadmap: machine-learning
node: generative-models
group: deep-learning-frontiers
video:
  provider: youtube
  id: "8L11aMN5KY8"
  start: 0
  end: null
creator:
  name: Luis Serrano Academy
  channel_url: https://www.youtube.com/@SerranoAcademy
duration_seconds: 1261
level: advanced
prerequisites: [training-neural-networks]
takeaways:
  - An autoencoder compresses an input into a small representation and then reconstructs the original from that compression, learning what is genuinely essential about the data in the process.
  - A generative adversarial network trains two competing networks at once — a generator producing fake data and a discriminator trying to tell fake from real — each improving because of the other's pressure.
  - Both approaches generate new data rather than predicting a label for existing data, which is a fundamentally different task from everything classification- and regression-focused covered earlier in this roadmap.
check_questions:
  - q: What does an autoencoder's compressed middle layer actually represent?
    a: A compact summary of whatever the network found genuinely essential about the input, discovered by the pressure of having to reconstruct the original from only that small summary — information the network found dispensable gets discarded, since keeping it would not have been necessary to reconstruct the input well.
  - q: Why does training a GAN involve two networks improving each other rather than one network training alone?
    a: The generator only gets better at producing convincing fakes by trying to fool an increasingly capable discriminator, and the discriminator only gets better at spotting fakes by studying an increasingly convincing generator. Neither one improves in isolation — each one's progress is driven directly by the other's.
deeper:
  - label: "IBM Technology — What are Autoencoders?"
    url: https://www.youtube.com/watch?v=qiUEgSCyY5o
  - label: "IBM Technology — What are GANs?"
    url: https://www.youtube.com/watch?v=TpMIssRdhco
related_formulas: []
---

## A different kind of task entirely

Every model covered earlier in this roadmap predicts something about existing
data — a label, a continuous value, a cluster assignment. Generative models
do something categorically different: they produce new data that resembles
what they were trained on, rather than making a prediction about data that
already exists. A model that generates a plausible new face image, or a
synthetic but realistic medical record for testing purposes, is doing
generative modelling, and the two approaches covered in this lesson represent
two genuinely different ways of achieving it.

## Autoencoders: compress, then reconstruct

An **autoencoder** is trained on a deceptively simple task: take an input,
compress it down through a narrow middle layer far smaller than the input
itself, then reconstruct the original input as closely as possible from only
that narrow compressed representation. The network is never told what to
keep in that compressed middle layer — it discovers this on its own, purely
from the pressure of needing to reconstruct the original well. Whatever
survives into that compressed representation is, by construction, whatever
the network found genuinely essential about the input; whatever gets
discarded was redundant enough that the reconstruction succeeded without it.

This compressed representation is useful well beyond reconstruction itself —
it is a learned, compact summary of the input that can be used for anomaly
detection (a genuinely unusual input reconstructs poorly, since the network
never learned a good compression for anything like it), for denoising (train
the reconstruction target to be a clean version of a noisy input), or simply
as a reduced-dimensionality representation in its own right, conceptually
related to the
[dimensionality reduction](/roadmaps/machine-learning/dimensionality-reduction) covered earlier in this roadmap, though arrived at through a
neural network's training rather than through PCA's eigenvector mathematics.

## GANs: two networks, locked in competition

A **generative adversarial network (GAN)** trains two networks simultaneously
and in direct opposition. The **generator** takes random noise as input and
tries to produce output that looks like genuine data from the training set.
The **discriminator** is shown a mix of genuine training examples and the
generator's output, and tries to correctly tell which is which. These two are
trained together: the generator improves by learning to fool an increasingly
capable discriminator, while the discriminator improves by learning to catch
an increasingly convincing generator. Neither one can get meaningfully better
in isolation — each one's progress is driven directly by the pressure the
other one applies, and training continues until the generator produces output
convincing enough that the discriminator can do no better than random
guessing at telling it apart from the real thing.

## Two different routes to the same destination

Autoencoders and GANs solve the generative problem in genuinely different
ways. An autoencoder is trained with a direct, explicit reconstruction target
— there is always a specific correct output to measure against during
training. A GAN has no such direct target at all; its generator improves only
indirectly, through the discriminator's evolving feedback. This difference in
training signal is a large part of why the two approaches tend to produce
output with different characteristic strengths and weaknesses, and why both
remain in active, separate use today rather than one having fully
superseded the other.
