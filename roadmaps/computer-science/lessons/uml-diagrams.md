---
title: "UML Diagrams"
description: Class, use-case, activity, state-machine, and sequence diagrams — what each one shows and when to reach for it.
roadmap: computer-science
node: uml-diagrams
group: design-patterns-and-architecture
video:
  provider: youtube
  id: "OkC7HKtiZC0"
  start: 0
  end: null
creator:
  name: Derek Banas
  channel_url: https://www.youtube.com/@derekbanas
duration_seconds: 767
level: beginner
prerequisites:
  - gof-design-patterns
takeaways:
  - UML (Unified Modeling Language) is a shared set of diagram conventions for describing a software system's structure and behavior without relying on any one programming language's syntax.
  - A class diagram shows static structure — classes, their fields and methods, and the relationships between them (inheritance, composition, association) — and is the closest UML diagram to actual code.
  - A use-case diagram stays at the level of "who does what with the system," showing actors (users or other systems) connected to the use cases (goals) they can trigger, without describing how those goals get implemented.
  - An activity diagram reads like a flowchart for a process or algorithm, showing the order of steps, decision points, and parallel paths a workflow can take.
  - A sequence diagram is the one diagram that shows time explicitly — vertical lifelines for each participant, with horizontal arrows showing which message gets sent to which object in what order — which makes it the right tool for understanding or documenting a specific interaction, not the system as a whole.
check_questions:
  - q: What's the difference between what a class diagram shows and what a use-case diagram shows?
    a: A class diagram shows the system's internal static structure — classes, their fields, methods, and relationships — while a use-case diagram stays outside the system entirely, showing which actors can trigger which goals without describing any implementation detail.
  - q: Why is a sequence diagram the right choice for documenting "what happens when a user clicks checkout," rather than a class diagram?
    a: A sequence diagram is built around time and message order — it shows exactly which object calls which other object's method, in what sequence, across a single scenario — while a class diagram only shows static structure and says nothing about the order operations happen in.
  - q: If you need to document the branching steps of an order-fulfillment process (check stock, then either ship or notify "out of stock"), which UML diagram fits best, and why?
    a: An activity diagram, because it's built specifically to show a sequence of steps with decision points and branching paths, the same way a flowchart would, rather than focusing on object structure (class diagram) or a single object-to-object interaction (sequence diagram).
deeper:
  - label: "How to Make a UML Sequence Diagram"
    url: "https://www.youtube.com/watch?v=pCK6prSq8aw"
related_formulas: []
---

## A shared vocabulary for describing software

Two engineers can write the exact same system in Java and Python and still want to agree on its design before either one writes a line of code. **UML**, the Unified Modeling Language, is the diagramming vocabulary that lets them do that — a set of conventions for boxes, arrows, and symbols that mean the same thing regardless of implementation language. UML has around a dozen diagram types in its full specification, but five cover the overwhelming majority of real use: class, use-case, activity, state-machine, and sequence diagrams. Each answers a different question about the same system, and picking the wrong one for the job is the most common way UML diagrams become confusing rather than clarifying.

## Class diagrams: structure, close to code

A **class diagram** is the one that looks most like the code it describes. Each class is a box with three compartments: the class name, its fields, and its methods. Lines between boxes show relationships — a solid line with a hollow triangle arrowhead means inheritance (`Dog` **is a** `Animal`), a line with a filled diamond means composition (a `Car` **owns** an `Engine`, and the engine doesn't outlive the car), and a plain line means a weaker association (a `Student` **is enrolled in** a `Course`, but neither owns the other). A class diagram for a small library system might show `Book`, `Member`, and `Loan`, with `Loan` associated to both `Book` and `Member` to represent "this member currently has this book." It's the diagram you'd sketch before writing the actual class definitions, or draw after the fact to onboard someone to an unfamiliar codebase's shape.

## Use-case diagrams: who does what, not how

A **use-case diagram** deliberately stays outside the system. It has **actors** — stick figures representing a user or an external system — connected by lines to ovals representing **use cases**, which are goals the actor can accomplish: "Borrow Book," "Return Book," "Search Catalog." A librarian actor might connect to "Check Out Book" and "Add New Title," while a member actor connects only to "Search Catalog" and "Borrow Book." The diagram says nothing about databases, classes, or algorithms — its entire value is forcing a conversation about scope and audience before any design work happens: who are the users, and what are they actually trying to do?

## Activity diagrams: flowcharts for process

An **activity diagram** looks like, and functions as, a flowchart. Rounded rectangles are actions, a diamond is a decision point with labeled branches, and solid bars represent a fork into parallel activities that later rejoin. For an order-fulfillment process: "Receive Order" flows into "Check Stock," which branches at a decision diamond into "Ship Order" if stock is available or "Notify Customer: Out of Stock" if it isn't. Where a use-case diagram answers "what can an actor do," an activity diagram answers "what are the steps, in what order, and where does the process branch" — it's the right tool once you're past "what are the goals" and into "how does this workflow actually unfold."

## State-machine diagrams: what state is this object in?

A **state-machine diagram** tracks one object's lifecycle as it moves between a finite set of **states**, with labeled arrows showing the **events** that trigger each transition. An `Order` object might have states `Pending`, `Paid`, `Shipped`, and `Cancelled`, with an arrow from `Pending` to `Paid` labeled `payment received`, and arrows from both `Pending` and `Paid` to `Cancelled` labeled `cancel requested` — but no arrow from `Shipped` to `Cancelled`, because the diagram is also documenting that a shipped order can no longer be cancelled. This is especially useful for objects with rules about which transitions are even legal, since the diagram makes illegal transitions visible by their absence.

## Sequence diagrams: the one with time in it

Every diagram so far describes structure or possibility; a **sequence diagram** is the one built around a single concrete scenario unfolding over time. Each participant — an object, a user, an external service — gets a vertical **lifeline** running down the page, and horizontal arrows between lifelines represent messages, read top to bottom as the sequence in which they happen. A "user checks out" sequence diagram might show the `Browser` lifeline sending `submitOrder()` to a `Checkout` object, which sends `chargeCard()` to a `PaymentGateway` lifeline, which replies with `chargeSuccessful`, after which `Checkout` sends `createOrder()` to an `OrderService`. Because it nails down exact message order for one specific interaction, a sequence diagram is the right tool when a bug report says "sometimes the payment succeeds but the order never gets created" — you draw out exactly what calls what, in what order, and look for the gap, which is exactly what the deeper video on building one works through step by step.

## Picking the right diagram

The five diagrams split cleanly by the question being asked: structure of the whole system (class), what users can accomplish (use-case), the steps of a process (activity), one object's lifecycle (state-machine), or the exact unfolding of a single interaction (sequence). Reaching for a class diagram to document a bug in message ordering, or a sequence diagram to capture an entire system's class structure, is a sign the wrong tool got picked for the question actually being asked.
