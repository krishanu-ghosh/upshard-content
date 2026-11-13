---
title: "Template Method and Chain of Responsibility"
description: Template Method fixes an algorithm's skeleton with overridable steps; Chain of Responsibility passes a request along handlers until one handles it.
roadmap: software-design-architecture
node: template-method-and-chain-of-responsibility
group: behavioral-patterns
video:
  provider: youtube
  id: "FafNcoBvVQo"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 456
level: intermediate
prerequisites: [command-and-state]
takeaways:
  - "Template Method puts an algorithm's fixed sequence of steps in a base class method, marks the steps that vary as abstract or overridable, and lets subclasses fill in only those steps — the order of operations itself is never duplicated or re-decided by a subclass."
  - "A `DataProcessor` base class can define `process()` as \"read, transform, write\" once, with `read()` and `write()` already implemented and only `transform()` left abstract — a `CsvProcessor` and a `JsonProcessor` each override just `transform()` and inherit the rest unchanged."
  - "Chain of Responsibility passes a request along a sequence of handler objects, each one deciding independently whether to handle it or forward it to the next handler in the chain, so the sender never needs to know which handler will end up acting."
  - "Middleware in a web framework is Chain of Responsibility in practice — an authentication check, a logger, and a rate limiter each get a turn at the request and can act, modify it, or pass it on, without any of them knowing what comes before or after."
  - "An approval chain that escalates — a manager who can approve small expenses, forwarding anything larger to a director, who forwards anything larger still to a VP — lets a new approval tier get added as a new handler spliced into the chain, with no existing handler rewritten."
  - "Both patterns avoid duplicating control flow: Template Method avoids re-writing the same step order in every subclass, and Chain of Responsibility avoids a single method with a long `if` chain deciding who handles what — but Template Method fixes the structure in one base class, while Chain of Responsibility lets the structure itself (the chain's order and membership) be assembled and changed at runtime."
check_questions:
  - q: What specifically does a subclass override in Template Method, and what does it never get to change?
    a: It overrides only the individual step methods the base class marked as abstract or overridable — like `transform()` in a `DataProcessor` — and it never changes the order those steps run in, because that sequence lives in one non-overridden method (often marked `final`) in the base class.
  - q: Why does a request-handling chain not need to know in advance which handler will process a given request?
    a: Each handler in the chain only checks whether the request matches what it's responsible for; if not, it forwards the request to the next handler and takes no further action, so the sender just hands the request to the first handler and the chain itself works out, request by request, which link actually handles it.
  - q: When would Chain of Responsibility be the wrong tool even though a request does need to pass through several checks?
    a: When every one of those checks always has to run, in a fixed order, with no handler ever "absorbing" the request and stopping it early — that's just a sequence of steps, which Template Method or a plain pipeline function expresses more directly than a chain built to support early exit and dynamic membership it never actually uses.
deeper:
  - label: "The Template Method Pattern Explained and Implemented in Java | Geekific"
    url: "https://www.youtube.com/watch?v=cGoVDzHvD4A"
related_formulas: []
---

## Fixing the shape of a process while leaving room to vary it

Both of these patterns exist because a process has a shape that stays constant while some part of it needs to differ — by subclass, in Template Method, or by which handler happens to be configured, in Chain of Responsibility. Neither pattern invents new behavior; both are about *where* varying behavior is allowed to live without duplicating the parts that don't vary.

## Template Method: the skeleton is fixed, the steps aren't

Imagine a `DataProcessor` base class whose job is "read data, transform it, write it out." Without this pattern, every format-specific subclass — `CsvProcessor`, `JsonProcessor`, `XmlProcessor` — ends up writing its own `process()` method that calls read, then transform, then write, in that order, every single time. The order itself gets copy-pasted into every subclass, and if you ever need to add a validation step between read and transform, you're editing every subclass's `process()` method to insert it correctly, hoping you don't miss one.

Template Method puts `process()` in the base class exactly once, usually marked so it can't be overridden, implemented as a fixed call sequence: `read(); transform(); write();`. `read()` and `write()` have reasonable default implementations in the base class already. `transform()` is left abstract. `CsvProcessor` overrides only `transform()` to parse comma-separated values into records; `JsonProcessor` overrides only `transform()` to parse JSON. Neither one touches `process()`, because the order "read, then transform, then write" was never theirs to decide — it belongs to the base class, decided once.

This is worth it exactly when the sequence of steps really is shared and only a specific piece legitimately varies. If every subclass needs a wildly different sequence — not just a different `transform()`, but a different *order* of operations — Template Method is fighting the problem instead of solving it; at that point each subclass just needs its own method, full stop.

## Chain of Responsibility: let the request find its own handler

Picture an expense-approval process: a manager can approve amounts under $500, a director under $5,000, a VP anything above that. The naive version is one method with `if (amount < 500) managerApprove(); else if (amount < 5000) directorApprove(); else vpApprove();` — and adding a new approval tier means editing that method and re-testing every branch that already worked.

Chain of Responsibility turns each approver into a `Handler` object with a `handle(request)` method and a reference to the next handler in line. The manager handler checks if the amount qualifies; if so, it approves and stops. If not, it calls `next.handle(request)`, passing the request down the chain without needing to know what "down the chain" even contains. The code that submits an expense just calls `chain.handle(request)` on the first handler — it has no idea, and doesn't need to, whether the manager, the director, or the VP ends up approving it. Inserting a new tier between director and VP means constructing the chain with one more handler in the right spot; no existing handler's code changes.

This is the same shape behind web framework middleware: an authentication check, a request logger, and a rate limiter each get a turn at an incoming request, each deciding independently whether to act, modify the request, short-circuit it, or pass it along.

The pattern stops paying for itself when every step in the sequence always runs, in a fixed order, with nothing ever "absorbing" the request early — that's a plain pipeline, and dressing it up as a chain of handlers adds objects and indirection for a dynamic-membership, early-exit flexibility you're never actually going to use.

## Two different places to put "what varies"

Template Method nails the process's shape down in one base class and opens exactly one window — the overridable step — for variation. Chain of Responsibility doesn't fix anything: the chain's length, order, and membership can all change at runtime, and "what varies" is which handler, if any, decides to act. If your process has one correct order of operations with a single customizable step, reach for Template Method. If your process is really "try these in sequence until something claims it," reach for Chain of Responsibility.
