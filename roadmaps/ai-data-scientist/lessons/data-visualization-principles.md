---
title: Data Visualization Principles
description: What makes a chart actually communicate — choosing the right form, cutting chartjunk, and avoiding visuals that mislead.
roadmap: ai-data-scientist
node: data-visualization-principles
group: exploratory-data-analysis
video:
  provider: youtube
  id: "XgY1-BsZSzM"
  start: 0
  end: null
creator:
  name: storytelling with data
  channel_url: https://www.youtube.com/@storytellingwithdata
duration_seconds: 631
level: beginner
prerequisites: [data-cleaning-and-missing-data]
takeaways:
  - A chart's form should follow the comparison you want the reader to make — part-to-whole, change over time, ranking, or correlation each has a form that makes that specific comparison easy and others that make it hard.
  - Chartjunk — heavy gridlines, 3D effects, decorative icons, redundant legends — doesn't add information, it adds work the reader's eye has to filter out before reaching the data.
  - A truncated y-axis, a dual-axis chart with mismatched scales, and area-as-value in a bubble chart are three of the most common ways a chart misleads without anyone intending it to.
  - The same underlying data can be shown honestly or dishonestly, so chart choice is a judgment call about faithfulness, not just a stylistic preference.
check_questions:
  - q: Why does the "right" chart type depend on the comparison you're making?
    a: Because each chart form makes one kind of visual judgment easy at the expense of others — a pie chart makes part-to-whole easy to see but ranking hard, while a bar chart does the reverse, so picking the form means first deciding which comparison actually matters to the reader.
  - q: What is chartjunk, and why is it a problem even when it looks polished?
    a: Chartjunk is any visual element that doesn't carry data — heavy gridlines, 3D bevels, decorative icons, unnecessary legends — and it's a problem because the reader's eye has to spend effort filtering it out before it can find the actual values, even if the chart looks professionally designed.
  - q: How does a truncated y-axis mislead a reader, and what's the usual fix?
    a: It exaggerates the visual size of a difference between bars or points by stretching a narrow range of the data across the full height of the chart, and the usual fix is to start a bar chart's axis at zero, or to clearly annotate a zoomed-in axis so the reader knows it isn't starting at zero.
deeper:
  - label: "How to Choose the Best Chart for Your Data Visualization"
    url: "https://www.youtube.com/watch?v=fBgk_cNCnsk"
  - label: "Misleading Data Visualisation and Best Charting Practices"
    url: "https://www.youtube.com/watch?v=yntLglYUHXI"
related_formulas: []
---

## Start with the comparison, not the chart type

A common mistake is picking a chart type first and forcing the data into it. The better order is to decide what comparison you want the reader to make, and let that decide the form. There are a handful of comparison types that cover most real work: change over time (a line chart), ranking or magnitude across categories (a bar chart), part-to-whole (a stacked bar or, sparingly, a pie chart), correlation between two numeric variables (a scatter plot), and distribution of a single variable (a histogram or box plot). Each form is good at exactly one or two of these and bad at the rest. A pie chart makes "roughly a third of the total" easy to see at a glance but makes it nearly impossible to tell whether one slice is 22% or 24% of the total — human eyes are much better at comparing the lengths of bars than the angles of wedges. If the real question is "which product sold the most," a sorted bar chart answers it in half a second; the same data in a pie chart forces the reader to do mental arithmetic the chart should have done for them.

## Line charts, bar charts, and scatter plots in practice

Line charts imply continuity, so they belong to data that's genuinely continuous over an ordered axis, almost always time. Using a line chart to connect unordered categories (say, five unrelated product names) implies a trend between them that doesn't exist and actively misleads. Bar charts are the default workhorse for comparing discrete categories, and they carry an important convention: their length encodes the value, so a bar chart's axis should start at zero, or the lengths stop being comparable at a glance. Scatter plots exist to show the relationship between two numeric variables and are the right first chart to reach for when you suspect — but haven't confirmed — a correlation; adding a third variable through point color or size turns a scatter plot into a compact multivariate view, but each added encoding (color, size, shape) raises the cognitive cost of reading it, so use more than two or three only when the audience genuinely needs that density.

## Cutting chartjunk

Edward Tufte's term "chartjunk" describes any visual element that doesn't encode data but still asks for the reader's attention: heavy 3D bevels on bars, gratuitous drop shadows, dense gridlines behind every value, redundant data labels stacked on top of an axis that already shows the same numbers, or decorative clip-art icons scattered around a plot. None of this is neutral — it costs the reader real effort to mentally subtract the decoration before they can extract the actual values, and 3D effects specifically distort the reader's perception of relative size because the rendering angle changes how large a bar or wedge appears to be, independent of its real value. The fix is almost always subtraction rather than addition: remove gridlines until only the ones that help orientation remain, drop a legend if you can label series directly on the chart instead, and resist adding color unless color is doing actual work (encoding a category or highlighting one specific series), because color used purely for decoration pulls attention away from the data it's meant to serve.

## How charts mislead, intentionally or not

Beyond chartjunk, there are specific techniques that distort what a chart communicates even when nobody involved meant to deceive. A truncated y-axis — starting a bar chart at 80 instead of 0 — turns a 2% real difference into a bar that looks three times taller, because the human eye reads bar length, not the axis labels. Dual-axis charts, where two series share a plot but each has its own independently-scaled y-axis, can make two unrelated trends appear to move together purely because of how the two scales happen to be set, a classic way to imply a causal or correlated relationship that the raw numbers don't support. Bubble charts encode a third variable in a circle's area, but many tools (and many chart-makers) scale a circle's *radius* linearly with the value instead of its area — since area grows with the square of radius, this makes differences look far larger than they are, because perceived size is driven by area, not radius.

## Exploratory versus presentation visuals

It's worth distinguishing a chart you make for yourself while exploring data from one you make for someone else to read. Exploratory charts can be messy, fast, and numerous — you might generate twenty histograms in a notebook just to get a feel for a dataset, and clarity for an audience of one (you, with full context) matters less than speed. Presentation charts need the opposite: fewer charts, each one earning its place, every element justified, and the title doing real work (a title like "Revenue grew 18% in Q3, driven by the enterprise segment" tells the reader what to look for before they even look at the data, which a generic title like "Q3 Revenue" never does). The skill isn't making every chart beautiful; it's knowing which stage you're in and matching the effort to it.
