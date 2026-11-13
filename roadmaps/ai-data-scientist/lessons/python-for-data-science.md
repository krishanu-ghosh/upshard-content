---
title: Python for Data Science
description: "The Python idioms that matter for data work: records as dicts, functions over copy-pasted loops, and reading files without losing your afternoon."
roadmap: ai-data-scientist
node: python-for-data-science
group: data-science-coding
video:
  provider: youtube
  id: "HrRA67O-QXI"
  start: 0
  end: null
creator:
  name: Visual Studio Code
  channel_url: https://www.youtube.com/@code
duration_seconds: 4352
level: beginner
prerequisites: [data-scientist-vs-ml-engineer-vs-analyst]
takeaways:
  - "A list of dicts is the mental model for tabular data before you ever load a dataframe: each dict is a row, each key is a column."
  - A function you can call with different arguments replaces five near-identical copy-pasted blocks and is where most scripting bugs stop hiding.
  - "Reading files line by line and closing them properly (via `with open(...)`) avoids the two most common ways a data script silently loses or corrupts data."
  - List comprehensions are for building a new list from an existing one in one readable line, not for cramming unrelated logic into a single statement.
check_questions:
  - q: "Why is a list of dictionaries a useful mental model for tabular data, even before you use pandas?"
    a: "Each dictionary acts like a row with named fields, and a list of them acts like a table, which maps directly onto what a dataframe or a database table actually is under the hood."
  - q: "What's the practical reason to write a function instead of copy-pasting a loop body with small changes?"
    a: "A function turns the repeated parts into one place to fix bugs and the changing parts into explicit arguments, so a mistake found later only has to be corrected once instead of in every copy."
  - q: "Why use `with open(...)` instead of calling `open()` and `close()` separately?"
    a: "The `with` block guarantees the file gets closed even if an error happens while you're reading it, which prevents the silent data loss and locked-file bugs that come from a file never getting closed."
deeper:
  - label: Python for Data Science - Course for Beginners
    url: "https://www.youtube.com/watch?v=LHBE6Q9XlzI"
related_formulas: []
---

## Records as dictionaries, tables as lists

Before you touch pandas, it helps to see what a dataframe actually is: rows and columns. In plain Python, the closest match is a dictionary for a row and a list of dictionaries for a table. A dictionary like `{"name": "Mara", "age": 29, "city": "Austin"}` is a record with named fields, exactly like a row in a spreadsheet. A list of several such dictionaries, `rows = [{"name": "Mara", ...}, {"name": "Sam", ...}]`, is a small table. This matters because most of the operations you'll later do with pandas — filter rows, pull one column, compute a total — have a direct, if clumsier, Python equivalent: `[r["age"] for r in rows if r["city"] == "Austin"]` filters and extracts in one line. Building this mental model first means pandas later feels like a faster way to do something you already understand, not a block box you're memorizing syntax for.

The companion structure is the dictionary used for counting or grouping. If you want to tally how many rows come from each city, you loop once and accumulate into a dict: `counts = {}`, then for each row `counts[row["city"]] = counts.get(row["city"], 0) + 1`. That `.get(key, default)` pattern — "give me this key's value, or 0 if it's not there yet" — shows up constantly in data scripts and is worth memorizing on its own, because the alternative (checking `if key in counts` first) is two lines where one will do.

## Functions over copy-pasted loops

A data script usually starts as a handful of loops hacked together to answer one question, then grows because you need the same logic on a different file, or with a different threshold. The moment you find yourself copying a block and changing one number inside it, that's the signal to extract a function. Write `def clean_price(raw):` once, with the parsing, stripping, and type conversion inside it, and call `clean_price(row["price"])` everywhere instead of retyping the logic. The benefit isn't just fewer lines — it's that when you discover the price field sometimes has a stray dollar sign, you fix it in exactly one place instead of hunting down every copy.

Functions also make scripts testable in a lightweight way: you can call `clean_price("$12.50")` directly in a notebook cell or a quick script and see the output, without running your entire pipeline. Default arguments are useful here too — `def clean_price(raw, currency_symbol="$")` lets you reuse the function for a dataset in a different currency without touching its body. Keep functions focused on one transformation each; a function that cleans, filters, and aggregates in one body is hard to reuse and hard to debug when the output looks wrong, because you can't tell which step broke.

## Reading and writing files without losing data

Nearly every data science task starts with a file on disk. The idiom that matters is `with open("data.csv") as f:` — the `with` block hands you the open file as `f` and guarantees it gets closed automatically, even if your code raises an error halfway through reading it. Compare that to calling `f = open(...)` and `f.close()` as separate lines: if an exception happens between them, the file is left open, which on a long-running script can exhaust your system's file handles or leave writes half-flushed to disk.

For plain text, `for line in f:` reads one line at a time instead of loading the whole file into memory, which matters once a file is bigger than a few megabytes. For structured data, Python's built-in `csv` module (`csv.DictReader(f)`) gives you each row as a dictionary keyed by the header row, which connects directly back to the list-of-dicts model from the first section. Writing works the same way: `with open("out.csv", "w") as f:` paired with `csv.DictWriter`. A common mistake is opening a file in text mode and writing numbers without converting them to strings first, or forgetting `newline=""` on Windows, which causes blank lines between every row in the output CSV — both are quick fixes once you know to look for them.

## Comprehensions, scripting habits, and where not to overdo it

List comprehensions — `[clean_price(r["price"]) for r in rows]` — are Python's way of saying "build a new list by transforming each item of an existing one," and they're genuinely more readable than the equivalent three-line loop once you're used to the pattern. They start to hurt readability when you nest them or add complex conditions; at that point, a regular loop with a named function reads better, and a reviewer six months from now (possibly you) will thank you for it.

Two scripting habits pay off disproportionately. First, guard your script's entry point with `if __name__ == "__main__":` so a file can be both imported (reusing its functions elsewhere) and run directly, without re-running the whole script on import. Second, print or log intermediate shapes and counts as you go — `print(len(rows), "rows loaded")` — rather than running a long pipeline blind and only inspecting the final output. When something's wrong with your data, these checkpoints tell you which step introduced the problem instead of leaving you to bisect the whole script by hand.
