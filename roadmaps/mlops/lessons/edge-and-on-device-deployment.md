---
title: Edge and on-device deployment
description: Running a model on the device itself trades accuracy and model size for no network dependency, lower latency, and better privacy.
roadmap: mlops
node: edge-and-on-device-deployment
group: serving-deployment
video:
  provider: youtube
  id: "Zg0t3f90n6Q"
  start: 0
  end: null
creator:
  name: TensorFlow
  channel_url: https://www.youtube.com/@TensorFlow
duration_seconds: 706
level: advanced
prerequisites: [deployment-strategies]
takeaways:
  - On-device deployment removes the network round-trip entirely, which is the only way to guarantee low latency and offline availability that server-based serving can't match.
  - Quantization and other compression techniques shrink a model to fit a device's memory and compute budget, at a real but often small cost to accuracy.
  - The decision to deploy on-device is usually driven by a hard constraint — privacy, connectivity, or latency — not by a general preference for one architecture over another.
check_questions:
  - q: Why does on-device inference have a lower latency floor than even a well-scaled server?
    a: Because it removes the network round-trip entirely, so the only remaining latency comes from the device's own compute, not from physics and network conditions a server can never fully eliminate.
  - q: What does quantization actually do to a model?
    a: It reduces the numerical precision used to store and compute the model's weights, such as converting 32-bit floating-point numbers to 8-bit integers, shrinking the model's size and compute cost at some cost to accuracy.
  - q: Why would a team choose on-device deployment even though it's harder to update than a server-side model?
    a: A privacy requirement that user data never leave the device, or a need for the feature to keep working with no network connection at all, can make on-device deployment the only option regardless of the update-management cost.
deeper:
  - label: "TensorFlow Lite: Solution for running ML on-device (TF World '19) — TensorFlow"
    url: "https://www.youtube.com/watch?v=0SpZy7iouFU"
related_formulas: []
---

## A fundamentally different deployment target

Everything covered so far in this group — serving patterns, rollout
strategies, scaling levers — assumes a model running on a server you
control, reachable over a network. Edge and on-device deployment throws
that assumption out: the model runs directly on the end-user's device —
a phone, a smart speaker, a car's onboard computer, an industrial sensor
— with no server in the loop at inference time at all. That's not a minor
variation on server-based serving; it's a different deployment target with
its own constraints, and understanding why it's chosen (or avoided) means
understanding what you gain and what you give up.

## What you gain: no network, no server

The most immediate benefit is removing the network round-trip entirely.
A server-based model, no matter how well it's scaled and cached, still
has to send a request out, wait for a response, and receive it back — and
that round-trip has a latency floor set by physics and network conditions
that no amount of server-side optimization eliminates. A model running
on-device responds as fast as the device's own hardware allows, with
nothing in between. For a feature like a camera app detecting objects in
a live viewfinder, or a voice assistant processing a wake word, that
round-trip latency isn't just inconvenient — it can make the feature
unusable.

On-device deployment also keeps working with no connectivity at all,
which matters for anything that has to function on a subway, in a remote
area, or in an industrial setting without reliable networking. And it
keeps the user's raw data on the device: a model that processes a photo
or an audio clip locally never has to transmit that raw data to a server
to get a prediction, which is a meaningfully stronger privacy guarantee
than "we deleted it after processing," because the sensitive data never
left the user's control in the first place.

## What you give up: size, accuracy, and easy updates

None of this comes free. A phone or embedded device has a tiny fraction
of a server's memory, compute, and power budget, and a model trained
without that constraint in mind usually doesn't fit. Getting it to fit
means compression — most commonly **quantization**, which reduces the
numerical precision used to represent the model's weights, for instance
converting 32-bit floating-point numbers down to 8-bit integers. This can
shrink a model's size by roughly 4x and substantially speed up inference
on hardware that handles low-precision math efficiently, but it does so
by discarding precision the model was trained with, which costs some
accuracy. The practical question is never "does quantization lose
accuracy" — it does — but "does the accuracy loss stay inside what the
feature can tolerate." A quantized keyword-spotting model that's 99.1%
accurate instead of 99.4% is usually a fine trade for fitting on a
microcontroller; a quantized medical-diagnosis model might not be.

Deployment itself also gets harder to manage. A server-side model update
is a rollout you control end to end — you can canary it, watch metrics,
and roll back in minutes. An on-device model update has to ship through
an app update or an over-the-air update mechanism, reach users who may
not update promptly, and in many cases can't be instantly rolled back
once it's on millions of devices. That operational gap is real, and it's
part of why teams don't default to on-device deployment just because it's
technically possible — they reach for it when a specific constraint
(latency floor, offline requirement, or privacy guarantee) makes
server-based serving actually unworkable, not merely less convenient.

## Choosing on-device deliberately

The decision isn't "edge is more advanced, so prefer it" — it's the
opposite instinct from the other lessons in this group. Server-based
serving should usually be the default, because it's easier to monitor,
update, and scale. On-device deployment earns its place when one of those
hard constraints — no tolerable network latency, no guaranteed
connectivity, or data that genuinely cannot leave the device — makes the
server-based tradeoffs from the rest of this group irrelevant, because
there's no server call to batch, cache, or canary in the first place.

## Carrying this forward

This closes out the serving-deployment group by showing the edge case,
literally: once compute moves onto the device, most of the server-side
toolkit — autoscaling, blue-green rollouts, request batching — simply
doesn't apply, and the real engineering work shifts to compression,
device compatibility, and update distribution instead. Recognizing which
world you're in, server or device, is what determines which half of this
group's lessons are even relevant to the system you're building.
