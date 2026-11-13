---
title: "Separation of Concerns, Coupling, and Cohesion"
description: What a "concern" is, why mixing two of them in one module causes pain later, and coupling and cohesion as the measurable form of the same idea.
roadmap: software-design-architecture
node: separation-of-concerns-and-coupling-and-cohesion
group: design-principles
video:
  provider: youtube
  id: "plMttQWztRM"
  start: 0
  end: null
creator:
  name: Modern Software Engineering
  channel_url: https://www.youtube.com/@ModernSoftwareEngineeringYT
duration_seconds: 812
level: beginner
prerequisites: [dry-kiss-and-yagni]
takeaways:
  - "A \"concern\" is a single reason a piece of code would need to change — parsing a date format and displaying that date are two concerns, even though they often sit next to each other in the same function."
  - "Mixing two concerns in one module doesn't cause a bug today; it causes pain the first time one concern needs to change without the other, and the code has no seam to let that happen cleanly."
  - "Coupling measures how much one module knows about, or depends on, the internals of another — low coupling means you can change one module's internals without touching its callers."
  - "Cohesion measures how tightly the responsibilities inside a single module belong together — high cohesion means everything in the module changes for the same reason."
  - "Coupling and cohesion are two sides of the same measurement: a module with low internal cohesion usually achieves it by reaching out and coupling tightly to other modules to get its job done."
  - "Separation of concerns is the design goal; coupling and cohesion are the two numbers that tell you whether you actually achieved it or just moved the mixing somewhere less visible."
check_questions:
  - q: What makes something a separate "concern" rather than just a separate step in the same task?
    a: A concern is defined by why it would change, not by where it sits in the code — parsing input and rendering output are separate concerns because a new input format and a new output format are two unrelated reasons to edit the code, even if today they're two lines next to each other in one function.
  - q: How can a module have low coupling but still be a badly designed module?
    a: Low coupling only means other modules don't depend on its internals — it says nothing about whether the module's own internals belong together; a module that does five unrelated things but hides them all behind one clean interface is low-coupling and low-cohesion at the same time, and the five unrelated things will still conflict with each other during a change.
  - q: Why does mixing concerns often go unnoticed until much later?
    a: The code works and passes its tests the day it's written, because nothing forces the two concerns to change independently yet — the cost only shows up the first time one concern has a reason to change without the other, and the two are tangled together closely enough that touching one risks breaking the other.
deeper:
  - label: "Highly COHESIVE Software Design to tame Complexity"
    url: "https://www.youtube.com/watch?v=r0-GC3Y_OME"
related_formulas: []
---

## A concern is a reason to change, not a step in a process

A "concern" is any single responsibility that could change on its own timeline, for its own reason, independent of everything else in the code. It's tempting to think of concerns as steps — "first parse the input, then render the output" — but the real test is what would force each one to change. A new input format is one reason to change; a new output format is a completely different, unrelated reason. Those are two concerns even if, in the code as written, they're two lines sitting right next to each other inside the same function. Separation of concerns just means giving each of those reasons its own place to live, so that a change driven by one reason doesn't have to touch code that exists for a different reason.

## Why mixing concerns doesn't hurt today — only later

Consider a function called `renderUserProfile` that takes a raw JSON blob from an API, parses out the fields it needs, formats a date, and then builds an HTML string to return. It works. It's tested. Nobody notices a problem, because nothing is forcing the parsing logic and the rendering logic to change independently yet. The pain shows up the day the API's date format changes but the HTML output shouldn't — now you're editing a function whose name and tests are about rendering, to fix something that has nothing to do with rendering, and you risk breaking the HTML output while you're in there. Split the function into `parseUserProfile` (concern: understanding the API's data) and `renderUserProfileHtml` (concern: producing markup), and the date-format fix touches only the parser; the renderer, and anything that tests it, is untouched.

## Coupling: how much one module has to know about another

Coupling is the measurable version of "did you actually separate these." Two modules are tightly coupled when one has to know details about the other's internals to work correctly — not just its public interface, but how it's implemented. If `renderUserProfileHtml` reaches into the raw JSON blob itself to grab a field the parser forgot to expose, it's coupled to the API's shape, not just to the parser's output. Low coupling means `renderUserProfileHtml` only ever touches the clean object `parseUserProfile` hands it — the API could switch from JSON to XML tomorrow, and the renderer would never need to know.

## Cohesion: how well a module's own contents belong together

Cohesion looks inward instead of outward. A module has high cohesion when everything inside it serves one responsibility and changes for one reason. `parseUserProfile` is highly cohesive if every line in it is about extracting and validating fields from the API response — the moment it also starts formatting a display string or writing a log file in a specific format a dashboard expects, it's doing two unrelated jobs, and a change to the dashboard's expectations now risks breaking data parsing. Low cohesion often hides behind a clean-looking interface: a module can expose one simple public function while internally doing three or four unrelated things, which means it looks separated from the outside but is just as tangled as the original one-function version on the inside.

## The two measurements point at the same mistake from different angles

Coupling and cohesion aren't independent dials — they trade against each other. A module that lacks internal cohesion (it's doing unrelated things) often compensates by reaching out and coupling tightly to whatever other module has the piece of logic it's missing, because it was never given a clear, self-contained job to do. Conversely, a highly cohesive module — one thing, one reason to change — naturally needs a narrower, more stable interface to its neighbors, which is what low coupling actually looks like in practice. Separation of concerns is the design intention; coupling and cohesion are how you check, concretely, whether that intention actually landed in the code or just moved the mess somewhere less obvious.

## Where this stops paying off

Splitting every three-line function into its own module in the name of cohesion produces a codebase where understanding one feature means jumping through a dozen tiny files — that's over-separation, and it trades one kind of pain (tangled logic) for another (scattered logic). The judgment call is whether two pieces of logic have genuinely independent reasons to change. If they've never changed independently in the project's history and there's no real reason to expect they will, keeping them together is simpler and more honest than splitting them on principle alone.
