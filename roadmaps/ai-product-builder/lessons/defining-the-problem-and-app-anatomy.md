---
title: Defining the Problem and App Anatomy
description: Turning a vague idea into a testable problem statement, and learning the frontend, API, backend, database, and auth pieces you'll direct AI to build.
roadmap: ai-product-builder
node: defining-the-problem-and-app-anatomy
group: foundations
video:
  provider: youtube
  id: "vDXkpJw16os"
  start: 0
  end: null
creator:
  name: Y Combinator
  channel_url: https://www.youtube.com/@ycombinator
duration_seconds: 381
level: beginner
prerequisites: [what-is-an-ai-product-builder]
takeaways:
  - A good problem statement names a specific person, a specific pain, and how they cope today — not just a feature you feel like building.
  - Testing an idea before building it means talking to the people who'd have the problem, not asking friends if they "like" the idea.
  - A typical web app has five parts — frontend, API, backend, database, and auth — and each one is something you'll be directing an AI tool to produce.
  - The frontend is what a user sees and clicks; the backend holds the logic and rules; the API is the contract connecting the two.
  - The database persists data between visits, and auth is the layer that knows who's making each request — both are common places where AI-generated shortcuts cause real damage.
check_questions:
  - q: Why is "jumping straight to a solution" a trap when starting a new product idea?
    a: Because a solution built before the problem is pinned down tends to solve an imagined problem rather than a real one — you end up with working software nobody needed.
  - q: What's the difference between the frontend and the backend of a web app?
    a: The frontend is the interface a user sees and interacts with in their browser or app; the backend runs on a server, holding the business logic, rules, and processing that the frontend can't or shouldn't do itself.
  - q: What job does the API do between the frontend and backend?
    a: The API is the defined contract through which the frontend asks the backend for data or actions and gets a response back — it's the boundary that lets the two be built, and reasoned about, somewhat independently.
deeper:
  - label: "Frontend, API, Backend and Database explained"
    url: "https://www.youtube.com/watch?v=NzEYYemQ3_8"
related_formulas: []
---

## Starting from a problem, not a feature

Most product ideas arrive backwards: someone thinks of a feature first — a dashboard, an app, a clever integration — and only later tries to justify why anyone would use it. A sturdier starting point is the opposite order. Name a specific kind of person, name the specific thing that currently wastes their time or money or patience, and describe how they deal with it today without your product. If you can't fill in those three blanks concretely, you don't have a problem yet, you have an enthusiasm, and enthusiasm is a bad foundation for deciding what an AI tool should spend the next hour building.

This matters more, not less, in an era where building a prototype takes minutes instead of weeks. When the cost of building something drops, the cost of building the wrong thing stops being "time," and becomes "attention spent validating a thing nobody asked for" — which is just as real a cost, and easier to miss because the tool made the building part feel free.

## Testing an idea before you build it

Validating a problem doesn't mean asking people whether they like your idea — people are agreeable, and agreeable answers are worthless. It means finding people who already have the problem you described and asking how they currently handle it, what they've already tried, and what it costs them to not have a better solution. If nobody has a workaround, a budget, or a complaint, the problem probably isn't painful enough yet to build for. This is tedious compared to generating a slick interface in a prompting tool, and it's also the single most valuable hour you'll spend, because it determines whether anything downstream is worth building at all.

## The anatomy of a web app

Once you have a real problem, you need a working vocabulary for what you're asking AI tools to produce, because "build me an app" decomposes into several distinct pieces with different jobs.

The **frontend** is everything a user actually sees and touches — the buttons, forms, and screens rendered in a browser or a phone app. It's responsible for layout, interaction, and making data understandable, but it generally shouldn't be trusted to enforce rules on its own, because anyone can open a browser's developer tools and tamper with it.

The **backend** is the program running on a server somewhere, invisible to the user, that holds the actual logic: calculating a price, checking whether an action is allowed, deciding what happens when a form gets submitted. This is where the rules that matter actually live, precisely because the backend is harder for an outside user to tamper with than code running in their own browser.

The **API** is the contract between those two — a defined set of requests the frontend is allowed to make ("get this user's orders," "create a new post") and the responses the backend promises to give back. Thinking of the API as a boundary is useful because it lets you reason about the frontend and backend as separate pieces that happen to agree on a shared language.

The **database** is where data survives between visits — user accounts, posts, orders, anything that needs to still exist after someone closes the tab and comes back tomorrow. Without one, your app has amnesia: every session starts from nothing.

**Auth** (short for authentication and authorization) is the layer that answers "who is making this request, and are they allowed to do that?" It's the piece most often shortchanged when moving fast, and the piece whose failure is most likely to leak someone else's data — which is exactly why it deserves deliberate attention rather than whatever an AI tool defaults to when you don't ask.

## Why this vocabulary matters before you prompt anything

None of this requires you to write these pieces by hand. It requires you to recognize them when an AI tool hands you a working app, so you can ask the right follow-up questions: where does this rule actually get enforced, what happens to this data after the page reloads, who's allowed to call this endpoint. Knowing the anatomy is what turns "it works" into "I understand why it works," which is the difference that matters once real users show up.
