---
title: Notebooks and Reproducible Analysis
description: "Using Jupyter notebooks well: avoiding hidden state from out-of-order execution, structuring cells, and versioning notebooks sanely."
roadmap: ai-data-scientist
node: notebooks-and-reproducible-analysis
group: data-science-coding
video:
  provider: youtube
  id: "lia4oIOPcG4"
  start: 0
  end: null
creator:
  name: JupyterCon
  channel_url: https://www.youtube.com/@JupyterCon
duration_seconds: 1428
level: intermediate
prerequisites: [pandas-and-numpy-workflows]
takeaways:
  - "A notebook's hidden state comes from its kernel's memory, not the order cells appear on the page, so a variable can exist from a cell you deleted or ran out of order and quietly poison later results."
  - "\"Restart kernel and run all\" is the single most important habit in notebook work: it's the only way to confirm a notebook actually reproduces its own results top to bottom."
  - Each cell should do one conceptual step (load, clean, transform, plot) so you can re-run and inspect a single stage without re-running the whole analysis.
  - "Committing a notebook's raw .ipynb to version control works, but stripping output before committing (via nbstripout or similar) keeps diffs readable and avoids leaking large embedded images into git history."
check_questions:
  - q: "A notebook cell references a variable that was never defined in any cell currently in the notebook, yet it runs without error. How?"
    a: "The variable was likely defined by a cell that was later deleted or edited, but the kernel's memory still holds the old value from when that cell last ran, so the notebook's visible code and the kernel's actual state have drifted apart."
  - q: "Why is \"restart kernel and run all\" considered the real test of whether a notebook works?"
    a: "It wipes the kernel's memory completely and re-executes every cell in the order they appear on the page, which is the only way to confirm the notebook produces its results from a clean state rather than from leftover variables created by cells you ran out of order during editing."
  - q: "Why strip cell outputs before committing a notebook to git, instead of committing it as-is?"
    a: "Outputs like embedded plot images are stored as the notebook's raw pixel data inside the JSON file, which makes diffs nearly unreadable and bloats the git history every time the notebook re-runs, even when the actual code didn't change."
deeper: []
related_formulas: []
---

## Where hidden state actually comes from

A notebook feels linear because cells are laid out top to bottom, but that's not how the underlying kernel works. The kernel is a running Python process with its own memory, and it doesn't care about the order cells appear on the page — it only cares about the order you actually executed them in. If you run cell 5, then go back and edit cell 2, then run cell 7 without re-running 2 through 6, the kernel's memory now reflects a sequence of executions that doesn't match what's visible on the page. Delete a cell that defined a variable, and that variable still lives in the kernel's memory until you restart it — later cells that reference it will keep working, giving you no signal that anything is wrong, right up until you (or someone else) opens the notebook fresh and the same cell fails with a `NameError`.

This is the single biggest source of "it worked on my machine" bugs in notebook-based analysis. The notebook you're looking at and the state actually producing your numbers can quietly diverge, and nothing in the interface tells you when that's happened. The execution count number next to each cell (`In [7]:`) is the only visible clue — if those numbers aren't in increasing order top to bottom, you've run cells out of sequence and should be suspicious of your current results.

## Restart and run all: the only real test

The fix is mechanical and non-negotiable for anything you intend to trust or share: before treating a notebook's output as final, use the "restart kernel and run all cells" command. This throws away all accumulated state and re-executes the notebook exactly as written, top to bottom, in a clean process. If it reproduces the same results, the notebook is actually self-contained. If it errors out partway through, or produces different numbers than what you'd been looking at, you've just caught a hidden-state bug before it reached a report or a stakeholder.

Make this a habit at natural checkpoints: before you save a notebook you'll come back to tomorrow, before you export a chart for a presentation, and always before declaring an analysis finished. It costs a few minutes and catches a category of bug that's otherwise nearly invisible until someone else tries to run your work and it doesn't match what you showed them.

## Structuring cells so they're worth debugging

A notebook where every cell does five things is hard to fix when something goes wrong, because you can't tell which of the five broke without picking the cell apart line by line. The useful discipline is one conceptual step per cell: a cell that loads data, a separate cell that cleans it, a separate cell that computes a derived column, a separate cell that plots it. When the plot looks wrong, you can re-run just the cleaning cell with a print statement added, inspect the output, and narrow down the problem without re-running everything upstream of it.

This also makes notebooks better analysis tools, not just safer ones. Keeping the cell that defines a helper function separate from the cells that call it means you can tweak the function and immediately re-run the three cells that use it, rather than re-running an entire pipeline for a one-line change. A loose convention worth adopting: imports and configuration at the top in one cell, then data loading, then a clear section (using markdown cells as headers) for each stage of the analysis, ending with a final cell that states the conclusion in plain language rather than leaving a reader to infer it from the last chart.

## Version control for notebooks

A `.ipynb` file is JSON under the hood, storing not just your code but the output of every cell, including embedded image data for any plots you generated. Committing this as-is to git works, but two things degrade over time: diffs become close to unreadable, because a one-line code change can rewrite large blocks of embedded binary-ish output data, and repository size balloons as plot images accumulate across commits, even when the underlying analysis code barely changed.

The standard fix is stripping outputs before committing, either with a tool like `nbstripout` configured as a git filter, or by manually clearing outputs (`Cell > All Output > Clear` in Jupyter, or `jupyter nbconvert --clear-output`) before saving. This keeps your git history focused on what actually changed — the code — and keeps diffs reviewable. The tradeoff is that your committed notebook no longer shows its own results when someone opens it on GitHub; if that matters for a specific analysis, some teams commit a parallel exported HTML or keep output in a designated "final" notebook while stripping output everywhere else. Either way, decide deliberately rather than letting git history quietly fill up with megabytes of chart images nobody will ever look at again.
