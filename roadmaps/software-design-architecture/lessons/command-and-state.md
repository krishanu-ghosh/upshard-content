---
title: "Command and State"
description: Command turns a request into an object with its own execute and undo; State replaces a sprawling conditional with one class per mode of behavior.
roadmap: software-design-architecture
node: command-and-state
group: behavioral-patterns
video:
  provider: youtube
  id: "UfGD60BYzPM"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 424
level: intermediate
prerequisites: [observer-and-strategy]
takeaways:
  - "Command wraps a request — an action plus the data it needs — in an object with an `execute()` method, instead of letting the caller invoke the receiver's method directly, which is what makes the request queueable, loggable, and undoable."
  - "A text editor's undo stack works because every edit is stored as a `Command` object with both `execute()` and `undo()` — undo doesn't \"figure out how to reverse\" an arbitrary action, it just calls the `undo()` the command already knew how to provide."
  - "A GUI button bound to a `Command` object can have its action swapped at runtime without touching the button's own code — the button only ever calls `command.execute()`, never the specific logic behind it."
  - "State says an object's behavior should change based on its internal mode, modeled as a family of state classes implementing a shared interface, rather than as one sprawling `if/switch` on a status field scattered through every method."
  - "A `TrafficLight` or an order-status machine built with State has one class per status — `RedState`, `GreenState`, `PendingState`, `ShippedState` — each implementing the same methods differently, and the object delegates to whichever state instance it currently holds."
  - "Command and State look similar on a class diagram — both wrap behavior in small classes behind an interface — but Command is about *when and how a single action runs*, while State is about *which of several behaviors is active right now*."
check_questions:
  - q: Why can't a plain method call support undo the way a Command object can?
    a: A plain method call executes an action and returns nothing durable — once it's run, there's no record of what happened to reverse; a `Command` object stores both the action and the data needed to undo it, so an undo stack just pops the last command and calls its own `undo()` instead of trying to reconstruct the previous state from scratch.
  - q: What's the problem with implementing a `TrafficLight` as one class with a `color` field and `if/else` statements in every method?
    a: Every method that behaves differently per color — `next()`, `canCarsGo()`, `display()` — ends up with its own copy of the same `if (color == RED) ... else if (color == YELLOW)` chain, so adding a new color or transition rule means finding and editing every one of those chains instead of changing code in one place.
  - q: How does State let you add a new mode of behavior without touching existing code?
    a: Since each mode is its own class implementing the shared state interface, adding a mode means writing one new class — a `FlashingState`, say — that implements the same methods the others do; the object holding a reference to "current state" and delegating to it never needs to change, because it was already written generically against the interface.
deeper:
  - label: "The State Pattern Explained and Implemented in Java | Geekific"
    url: "https://www.youtube.com/watch?v=abX4xzaAsoc"
related_formulas: []
---

## Turning an action and a mode into objects you can hold onto

Command and State share a habit: both take something that's normally implicit — "do this thing now," or "the object is currently in this mode" — and turn it into an actual object you can store, pass around, and swap out. That sounds like overkill until you hit the specific problem each one solves: undo, or a conditional that's grown unmanageable.

## Command: a request you can queue, log, or reverse

The ordinary way to handle a button click is to call a method directly — `saveButton.onClick(() => document.save())`. That's fine until you need to queue the action for later, log every action a user takes, or undo it. Command addresses all three by wrapping the request itself in an object: a `Command` interface with one method, `execute()`, and usually a second, `undo()`. A `PasteCommand` stores the text that was pasted and where; calling `execute()` inserts it, calling `undo()` removes it again. The button no longer calls `document.paste()` directly — it holds a reference to some `Command` and calls `command.execute()`, with no idea what concrete action is behind it.

This is exactly how a text editor's undo stack works. Every edit — type, delete, paste, format — gets pushed onto a stack as a `Command` object the moment it happens. Pressing Ctrl+Z pops the most recent command and calls its `undo()`. Nothing in the undo logic needs to know *what* the action was; it only needs every command to honor the same two-method contract. The same shape lets a command be queued for batch processing, serialized and sent over a network, or logged for an audit trail — anywhere "the action" needs to outlive the moment it was triggered.

Command isn't worth it for a one-off action with no need for undo, logging, or deferral — wrapping every function call in a class to satisfy a pattern name is ceremony without payoff. It earns its keep specifically when the *request* needs to be treated as data, not just invoked and forgotten.

## State: a conditional that becomes a family of classes

Consider a `TrafficLight` class with a `color` field and methods like `next()` and `canCarsGo()`. The naive version has `if (color == RED) ... else if (color == YELLOW) ... else if (color == GREEN)` inside *every* method that behaves differently per color. Add a flashing-yellow mode for a malfunction, and you're hunting down every one of those `if` chains to add a new branch — the transitions and the behavior are smeared across the whole class instead of living in one place per mode.

State fixes this by giving each mode its own class — `RedState`, `YellowState`, `GreenState` — all implementing the same interface (`next()`, `canCarsGo()`), and the `TrafficLight` holds a reference to whichever state instance is current, delegating every call to it. `canCarsGo()` on `TrafficLight` just becomes `return currentState.canCarsGo()`. An order-status system works identically: `PendingState`, `ShippedState`, `DeliveredState`, `CancelledState`, each knowing which transitions are legal from itself and refusing the rest. Adding `FlashingState` means writing one new class; nothing already working has to change.

The trade-off is the opposite of Command's: State is worth it exactly when the number of modes and the behavior differences between them are real and growing. A field with two states and one `if` doesn't need four classes and an interface — that's the sprawling-conditional cure applied to a conditional that was never actually sprawling.

## Not the same pattern wearing two names

It's easy to conflate these two, since both replace a bare-bones approach with small classes behind an interface. The test that separates them: Command is about *one action*, packaged so it can be stored, queued, logged, or reversed. State is about *one object's current mode*, modeled so behavior differs correctly across its lifetime without a conditional that has to be kept in sync everywhere. If you're asking "what should happen when this is triggered," you want Command. If you're asking "what's allowed given where this object currently is," you want State.
