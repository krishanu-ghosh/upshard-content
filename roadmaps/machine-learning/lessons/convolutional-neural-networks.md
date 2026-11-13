---
title: Convolutional neural networks
description: Filters sliding over an image, why this is far more parameter-efficient than a fully-connected layer, pooling, and the conv-pool-connected architecture.
roadmap: machine-learning
node: convolutional-neural-networks
group: neural-network-foundations
video:
  provider: youtube
  id: "HGwBXDKFk9I"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 923
level: advanced
prerequisites: [training-neural-networks]
takeaways:
  - A convolutional layer slides a small filter across an image, computing the same simple operation at every position, which detects a specific local pattern — an edge, a texture — wherever it appears in the image.
  - This is far more parameter-efficient than a fully-connected layer for image input, because one small filter's weights are reused at every position, rather than needing a separate weight for every pixel.
  - Pooling layers reduce the spatial size of the representation between convolutional layers, which both reduces computation and gives the network some tolerance to a pattern shifting slightly in position.
check_questions:
  - q: Why does a convolutional layer need far fewer parameters than a fully-connected layer processing the same image?
    a: A fully-connected layer needs a separate weight for every single connection between every input pixel and every output unit. A convolutional filter's weights are shared — the same small set of weights slides across every position in the image, so detecting an edge in the top-left corner uses exactly the same weights as detecting it anywhere else.
  - q: What does a pooling layer actually do to the data passing through it?
    a: It reduces the spatial dimensions of the representation, typically by taking the maximum (or average) value within small local regions, which both shrinks the amount of computation needed downstream and makes the network somewhat tolerant to a detected pattern appearing in a slightly different position than it did in training.
deeper:
  - label: "Computerphile — CNN: Convolutional Neural Networks Explained"
    url: https://www.youtube.com/watch?v=py5byOOHZM8
related_formulas: [convolution]
---

## Why images need a different kind of layer

A fully-connected layer, where every input connects to every output, treats
an image as an unstructured list of numbers — it has no concept that two
pixels sitting next to each other are more related than two pixels on
opposite corners. This throws away exactly the spatial structure that makes
an image an image, and it is also wildly expensive: a modestly sized image
already has many thousands of pixels, and a fully-connected layer would need
a separate weight for every single connection between every pixel and every
output unit. Convolutional layers were designed specifically to exploit image
structure instead of ignoring it.

## The convolution: one small filter, applied everywhere

A convolutional layer learns a small filter — commonly just three pixels by
three, or five by five — and slides it across every position in the image,
computing the same simple weighted-sum operation at each position; see the
[convolution](/formulas/convolution) formula page for the underlying
mathematics. A filter tuned to detect a vertical edge responds strongly
wherever a vertical edge appears in the image, regardless of whether that
edge sits in the top-left corner or the bottom-right. This is the key
efficiency gain over a fully-connected layer: the filter's weights are
**shared** across every position, so detecting the same pattern wherever it
appears in the image costs the same small number of weights rather than a
separate set for every location.

## Pooling: shrinking the representation on purpose

After a convolutional layer produces its output — one response map per
filter, showing where in the image that filter's pattern was detected — a
**pooling** layer reduces the spatial size of that output, most commonly by
taking the maximum value within small local regions (max pooling). This
serves two purposes at once: it reduces the amount of computation every
subsequent layer has to perform, and it gives the network some tolerance to a
detected pattern shifting slightly in position between one image and
another, since the pooled output stays the same as long as the strongest
activation remains somewhere within the pooled region, not at one exact pixel.

## The architecture, stage by stage

A typical convolutional network stacks several rounds of convolution followed
by pooling. Each successive round of filters operates on the previous
round's output rather than the raw image, learning to detect progressively
more complex patterns: early layers typically learn simple edges and
textures, deeper layers combine those into shapes, and eventually into
recognisable parts of whole objects. After several such rounds, the
resulting representation — now much smaller spatially but rich in detected
patterns — is flattened and passed into one or more ordinary fully-connected
layers, exactly the kind covered in
[neural network basics](/roadmaps/machine-learning/neural-network-basics).
Those layers combine everything detected so far into a final prediction.

## Why this group ends here and the next one begins

Convolutional networks are specifically well suited to data with strong
local, spatial structure — images above all, though the same filtering idea
extends to other grid-like data too. The next group in this roadmap turns to
a different kind of structure entirely: **sequences**, where what matters is
not spatial position but order over time, and where the right architecture
looks considerably different from anything covered in this group so far —
beginning with
[recurrent neural networks](/roadmaps/machine-learning/recurrent-neural-networks).
