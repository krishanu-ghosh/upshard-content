---
title: Dot products
description: Two vectors in, one number out — and why that number keeps appearing wherever similarity or projection is being measured.
roadmap: engine-preview
node: dot-products
group: the-maths-underneath
video:
  provider: youtube
  id: "LyGKycYT2v0"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 851
level: beginner
prerequisites: [vectors]
takeaways:
  - The dot product is the length of one vector times the length of the other's shadow on it.
  - Its sign alone answers "do these point the same way?" — positive, zero and negative mean aligned, perpendicular and opposed.
  - Projecting onto a line and multiplying by a one-row matrix are the same operation, which is why the dot product shows up in every layer.
check_questions:
  - q: Two unit vectors have a dot product of 0. What do you know?
    a: They are perpendicular. With unit length the dot product is exactly the cosine of the angle between them, and cos 90° is 0.
  - q: Why is the dot product the natural measure of similarity for embeddings?
    a: Because it grows with alignment and ignores nothing else — two embeddings pointing the same way score high however they were produced.
deeper:
  - label: Cosine similarity, and when to normalise first
    url: https://en.wikipedia.org/wiki/Cosine_similarity
related_formulas: [dot-product, cosine-similarity, vector-projection]
---

## The operation everything is built from

A dot product is the smallest useful thing you can do with two vectors: pair
them up, multiply, add. What makes it worth a node of its own is the second
description — it measures how much of one vector lies along the other — because
that is the description that explains why it appears in similarity search, in
attention scores, and in every single unit of a dense layer.

## Where this sits

Stack enough dot products side by side and you have matrix multiplication, which
is the next node. Much later, the same operation returns as the score between a
query and a key.
