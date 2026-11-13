---
title: Vectors
description: What a vector is from three directions at once — a list of numbers, an arrow in space, and a thing you can add and scale.
roadmap: engine-preview
node: vectors
group: the-maths-underneath
video:
  provider: youtube
  id: "fNk_zzaMoSs"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 591
level: beginner
prerequisites: []
takeaways:
  - A vector has three equally valid readings — a list, an arrow, and an element of a space where addition and scaling are defined.
  - Addition is tip-to-tail; scaling stretches or flips without changing direction.
  - Coordinates are not the vector; they are the vector written down in one particular basis.
check_questions:
  - q: Why does it matter that a vector can be read as a list and as an arrow?
    a: Because every later idea borrows one reading or the other — data is lists, geometry is arrows, and linear algebra is the dictionary between them.
  - q: What changes about a vector when you change basis?
    a: Its coordinates change; the vector itself does not. The arrow stays where it is while the ruler you measure it with moves.
deeper:
  - label: Essence of linear algebra — the full series
    url: https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab
related_formulas: [dot-product, euclidean-norm, vector-projection]
---

## Why this is the first node

Everything downstream of here — a layer of a network, a token embedding, an
attention head — is a vector operation wearing a different name. Starting
anywhere else means learning each of those as a separate trick instead of as
one idea applied three times.

## Where this sits

The next node turns two vectors into a single number, which is the operation a
neural network performs more often than any other. Nothing here requires
calculus yet — that arrives in its own node and the two meet at gradient
descent.
