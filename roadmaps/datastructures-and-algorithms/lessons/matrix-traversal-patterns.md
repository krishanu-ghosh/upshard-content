---
title: "Matrix Traversal Patterns"
description: Three recurring 2D-grid techniques — spiral traversal, in-place rotation, and flood-fill over connected regions — and how each handles boundaries.
roadmap: datastructures-and-algorithms
node: matrix-traversal-patterns
group: more-interview-patterns
video:
  provider: youtube
  id: "gCswsDauXPc"
  start: 0
  end: null
creator:
  name: Greg Hogg
  channel_url: https://www.youtube.com/@GregHogg
duration_seconds: 661
level: intermediate
prerequisites: []
takeaways:
  - "Spiral traversal is walked with four shrinking boundary variables (top, bottom, left, right rows/columns), not by trying to compute a direction-change formula — after finishing each side, move the corresponding boundary inward by one."
  - "90-degree in-place matrix rotation decomposes into two simpler, well-known operations: transpose the matrix (swap element [i][j] with [j][i]), then reverse each row — no second matrix is needed."
  - "Flood fill and 'number of islands'-style problems both start a BFS or DFS from every unvisited land cell, and the real content of the problem is how visited-tracking is done, not the traversal itself."
  - "You can often skip a separate visited array by mutating the grid in place — marking a visited land cell as water (or any clearly 'done' sentinel) as soon as you enter it — which is safe whenever the original grid doesn't need to survive the algorithm."
  - "All three patterns fail the same way when boundary checks are sloppy: spiral traversal over-counts or re-visits cells in non-square grids if a boundary isn't shrunk at the right moment, and flood fill infinite-loops or crashes without an in-bounds check before every recursive/BFS step."
check_questions:
  - q: "Why does spiral traversal use four separate boundary variables instead of just tracking a current direction and turning when you hit the edge of the array?"
    a: "Because 'the edge of the array' stops being the right stopping condition after the first lap — on later laps you need to stop at the edge of the *unvisited* region, which shrinks every time a side is completed; four boundaries that move inward after each side directly track that shrinking region, whereas a fixed array-edge check would re-visit already-completed cells."
  - q: "Why does transpose-then-reverse-rows correctly rotate a matrix 90 degrees clockwise, rather than just scrambling it?"
    a: "Transposing moves element [i][j] to [j][i], which turns each original row into a column; reversing each row of that transposed matrix then flips each of those new columns top-to-bottom, and the composition of 'turn rows into columns' plus 'flip each column' is exactly what a 90-degree clockwise rotation does to every element, which can be checked directly against the rotation's known effect on any corner cell."
  - q: "In a number-of-islands style problem, why is it safe to mark a visited land cell as water instead of maintaining a separate visited[][] array?"
    a: "Because once a cell has been visited, its only remaining job in the algorithm is to signal 'don't start a new search here and don't count me as unexplored land again' — marking it as water produces exactly that signal using the grid's own existing values, so a separate same-sized boolean array is redundant unless the original grid's content must be preserved after the algorithm finishes."
deeper:
  - label: "Spiral Matrix — LeetCode 54 (four-boundary walkthrough)"
    url: "https://www.youtube.com/watch?v=fcn8qkRcFVM"
related_formulas: []
---

## Why grids get their own pattern vocabulary

Arrays and linked lists have "fast and slow pointers" and "sliding window" as their recurring shapes; 2D grids have their own trio, because a grid adds a dimension that plain 1D traversal doesn't need to think about: direction. Spiral traversal is about *changing* direction in a disciplined order. Rotation is about *reinterpreting* positions without extra storage. Flood fill is about *spreading outward in all directions at once* and knowing when to stop. All three show up constantly because so many grid problems are really one of these three movements wearing a different story.

## Spiral traversal: four boundaries, shrinking inward

The trap in spiral traversal is trying to track "which direction am I currently moving" and "when do I turn" as a single piece of logic. It gets fiddly fast, especially on non-square grids. The clean version tracks four integers instead: `top`, `bottom`, `left`, `right`, initialized to the grid's actual edges.

Walk the top row left to right, then increment `top`. Walk the right column top to bottom, then decrement `right`. Walk the bottom row right to left — only if `top <= bottom` still holds, since the grid might have run out of rows — then decrement `bottom`. Walk the left column bottom to top, only if `left <= right` still holds, then increment `left`. Repeat until `top > bottom` or `left > right`.

For a 3×4 grid, after one full lap the four boundaries have each moved in by one, so the next lap only walks the single remaining middle row. Those shrinking boundaries are what make the algorithm naturally stop producing duplicates on irregular shapes.

## In-place rotation: transpose, then reverse rows

Rotating an n×n matrix 90° clockwise without allocating a second matrix looks tricky because it seems like you need to know every element's final destination simultaneously. The two-step decomposition sidesteps this entirely. First, transpose: for every `i < j`, swap `matrix[i][j]` with `matrix[j][i]`, turning rows into columns. Second, reverse every row in place. Walk `[[1,2,3],[4,5,6],[7,8,9]]` through it: transposing gives `[[1,4,7],[2,5,8],[3,6,9]]` (column 0 of the original is now row 0); reversing each row gives `[[7,4,1],[8,5,2],[9,6,3]]` — exactly the original grid rotated 90° clockwise. The fast sanity check is the corners: a clockwise rotation should send the original top-left corner (1) to the top-right corner, and it does.

## Flood fill and connected regions: search outward, mark as you go

"Number of islands" (count connected groups of land cells in a grid of 1s and 0s) and flood fill (recolor a connected region starting from one cell) are the same traversal: from a starting cell, explore every orthogonally adjacent cell that shares the relevant property (land, or the original color), recursively or via a BFS queue, and stop expanding when you hit water, a different color, or a grid boundary. The counting version wraps this in an outer scan: walk every cell of the grid, and whenever you find an unvisited land cell, that's a brand-new island — run the search from there (which will visit the entire island) and increment the count by one.

The detail that actually matters in interviews is visited-tracking. A separate `visited[][]` boolean grid works, but when the original grid's contents don't need to survive, it's simpler and more memory-efficient to mutate the grid directly: the instant you step onto a land cell, flip it to water (or any "already handled" sentinel) before recursing further. That single mutation does double duty — it prevents the outer scan from recounting the same island, and it prevents the inner search from revisiting cells and looping forever. Either way, every step — outer scan and inner search alike — needs an explicit bounds check (row and column within range) before touching a cell, since a flood fill without one will walk off the edge of the grid the moment it reaches a border cell.
