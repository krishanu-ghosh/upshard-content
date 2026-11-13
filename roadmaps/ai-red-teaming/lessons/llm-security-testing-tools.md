---
title: LLM security testing tools — garak, Promptfoo, and PyRIT
description: A practical tour of the field's most-used open-source LLM red-teaming tools and which one to reach for first, depending on the job.
roadmap: ai-red-teaming
node: llm-security-testing-tools
group: methodology
video:
  provider: youtube
  id: "YKyQeWJaLVM"
  start: 0
  end: null
creator:
  name: FranksWorld of AI
  channel_url: https://www.youtube.com/@FranksWorldTV
duration_seconds: 720
level: intermediate
prerequisites: [automated-vs-manual-red-teaming]
takeaways:
  - "garak scans a model broadly against a library of known probe categories, which makes it a good first pass for finding out what a model is already vulnerable to."
  - Promptfoo is built for developers who need red-teaming wired into a normal CI pipeline, turning a security scan into a report that runs on every change like a test suite.
  - PyRIT is an orchestration framework more than a fixed scanner — it gives you building blocks for composing custom, multi-step automated attack strategies.
  - These tools automate the known-pattern sweep that this roadmap's automated-testing lesson describes; they are not a replacement for manual testing's depth.
check_questions:
  - q: If you want a fast, broad first read on what categories of vulnerability a model already has, which tool fits best and why?
    a: Garak, because it is built as a vulnerability scanner that runs a model through a wide library of known probe categories and reports which ones succeed, similar in spirit to a general-purpose vulnerability scanner for a web application.
  - q: Why would a development team choose Promptfoo specifically over the other two tools?
    a: Promptfoo is designed to fit into a normal CI pipeline, so a team can run an LLM security and eval check automatically on every change and get a readable report, rather than running a one-off manual scan.
  - q: What makes PyRIT different in kind from garak, rather than just a competing scanner?
    a: PyRIT is an orchestration framework for composing custom automated attack strategies out of reusable building blocks, so it is suited to building a bespoke, multi-step attack rather than running one fixed, predefined set of probes the way garak does out of the box.
deeper:
  - label: "Promptfoo Red Teaming: A beginner's guide to finding vulnerabilities in Your LLM Apps — Jason Koo"
    url: https://www.youtube.com/watch?v=y6Dlsz5P8s8
  - label: "Episode 9: Automating Single-Turn Attacks with PyRIT | AI Red Teaming 101 — Microsoft Developer"
    url: https://www.youtube.com/watch?v=B9A9X6XMXOI
related_formulas: []
---

## Automation needs software, and the field has converged on a few

The previous lesson made the case that automated testing matters because it gives you breadth and repeatability manual testing cannot. That case only holds up if the tooling actually exists and is good. It does. AI red teaming has produced a small set of open-source tools that most practitioners reach for, each with a distinct niche. Knowing which one fits a given job is itself a practical skill, and it's the subject of this lesson.

## garak: broad vulnerability scanning

`garak`, maintained by NVIDIA, is built around the same basic idea as a conventional vulnerability scanner, just retargeted at language models. It organizes known attack and failure categories into "probes" — prompt injection, jailbreak patterns, data leakage, toxic output generation, and more. It runs a target model through each one and uses a "detector" to judge whether the response counts as a failure for that category. The output is a report showing where a given model currently stands across a wide spread of known issue categories. That makes `garak` a strong default for the first question any engagement asks: before doing anything custom, what does this model already fail at that the field already understands? Its probe library is broad rather than deep on any one category, so it's better suited to a wide first pass than to exhaustively stress-testing one specific weakness.

## Promptfoo: red teaming that lives in CI

`Promptfoo` takes a different angle, aimed at developers building LLM applications who want security testing to behave like the rest of their test suite. It's a CLI tool that runs both evaluation (does the model give good answers) and red-teaming (does it resist known attacks) as part of a normal development workflow — install it, point it at your application's prompts and configuration, run a scan, get a report meant for a developer to read, not just a security specialist. Its real value is in where it sits. Because it's CI-integrated, a team catches a regression — a prompt change that reopens an old jailbreak, a new feature that opens a fresh injection surface — automatically on every pull request, the same way it already catches a broken unit test. That makes `Promptfoo` the natural fit for ongoing, low-friction testing baked into how an application is built and shipped, rather than a standalone security engagement.

## PyRIT: a framework for building your own attacks

`PyRIT`, Microsoft's open-source Python Risk Identification Toolkit, isn't a fixed scanner with a built-in probe library the way `garak` is. It's an orchestration framework that gives you composable building blocks: prompt converters (for encoding or rephrasing an attack), attack strategies (single-turn, multi-turn, iterative refinement), targets (the model or system under test), and scoring logic for judging success. The tradeoff for that flexibility: `PyRIT` expects you to assemble a strategy rather than run one command and get a report. Reach for it when the other two don't fit — when you need to compose a custom, multi-step attack, like an adaptive jailbreak where one model iteratively refines prompts against another, or a strategy chaining several techniques from earlier in this roadmap in a way no off-the-shelf probe library anticipated.

## Picking a starting point

A reasonable rule of thumb: reach for `garak` first for a broad read on known weaknesses with minimal setup. Reach for `Promptfoo` when the goal is wiring red-teaming into a real development pipeline so regressions get caught automatically. Reach for `PyRIT` when the attack you need doesn't exist yet in a prebuilt library and has to be composed from primitives. None of the three replaces what manual testing finds — something genuinely new. They automate the known-pattern sweep the previous lesson described, each at a different point in a team's workflow.
