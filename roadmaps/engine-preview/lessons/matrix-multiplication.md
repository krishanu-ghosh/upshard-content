---
title: Matrix multiplication
description: Matrices as transformations, and multiplication as doing one transformation after another.
roadmap: engine-preview
node: matrix-multiplication
group: the-maths-underneath
video:
  provider: youtube
  id: "XkY2DOUCWMU"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 603
level: beginner
prerequisites: [vectors, dot-products]
takeaways:
  - A matrix is a transformation written down — its columns are where the basis vectors land.
  - Multiplying matrices composes transformations, which is why the order cannot be swapped.
  - The row-times-column rule is a consequence of composition, not the definition worth remembering.
check_questions:
  - q: Why is AB generally not BA?
    a: Because composition is ordered. Rotating then stretching lands somewhere different from stretching then rotating, and the matrices record exactly that.
  - q: What do the columns of a matrix tell you at a glance?
    a: Where each basis vector ends up after the transformation. Reading the columns is usually faster than computing anything.
deeper:
  - label: Linear transformations and matrices — the preceding chapter
    url: https://www.youtube.com/watch?v=kYB8IZa5AuE
related_formulas: [matrix-multiplication, determinant-2x2, trace]
---

## From a rule to a reason

Most people meet matrix multiplication as a rule about rows and columns, which
is memorable and explains nothing. The geometric reading — a matrix is a
transformation of space, and a product is one transformation followed by another
— turns the rule into something you can predict rather than recall.

## Where this sits

This is the last node before the network itself. A forward pass is a stack of
these transformations with a nonlinearity wedged between each pair, and the
reason the nonlinearity has to be there is that without it the whole stack
collapses back into one matrix.
