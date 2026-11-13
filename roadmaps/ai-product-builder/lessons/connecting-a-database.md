---
title: Connecting a Database
description: Moving an AI-generated prototype from mocked data to a real Postgres or Supabase database, and the common failure modes along the way.
roadmap: ai-product-builder
node: connecting-a-database
group: shipping-and-operating
video:
  provider: youtube
  id: "7kQi2HQMz4s"
  start: 0
  end: null
creator:
  name: Sara Dietschy
  channel_url: https://www.youtube.com/@saradietschy
duration_seconds: 2850
level: intermediate
prerequisites:
  - choosing-where-to-deploy
takeaways:
  - Most AI-generated prototypes start with mocked or in-memory data because it's fast to build against; that convenience is also exactly why it silently breaks the moment you need data to survive a restart or be shared between users.
  - Designing a real schema up front — what tables exist, what relates to what, what's required versus optional — avoids a much more painful retrofit later once real rows and real users depend on the current shape.
  - Connection strings and API keys belong in environment variables, never committed to the repository, and your AI coding agent needs to be told this explicitly or it will happily paste a secret straight into a file.
  - Switching from mock data to a real database usually surfaces hidden assumptions the prototype's code was making — that a field always exists, that a list is never empty, that an operation is instant — because mocked data was quietly well-behaved in ways real data isn't.
  - A managed Postgres provider like Supabase gives you a real relational database plus authentication and auto-generated APIs out of the box, which removes a lot of the plumbing that going from prototype to production used to require.
check_questions:
  - q: Why do AI-generated prototypes so often start with mocked or in-memory data instead of a real database?
    a: Mock data is faster to stand up and doesn't require any infrastructure decisions, so it lets the prototype demonstrate the app's behavior immediately — but it has no persistence, so data disappears on every restart and isn't shared across users.
  - q: What's the risk of skipping schema design and just letting the shape of your data emerge from the code?
    a: The data's shape ends up dictated by whatever the prototype happened to need first, and changing it later — once real rows exist and real features depend on the current structure — is a much more disruptive migration than designing the schema deliberately up front.
  - q: Where should a database connection string or API key live, and why does this matter more when an AI agent is writing your code?
    a: In an environment variable, never hard-coded into a committed file — an agent given a working connection string in context will often paste it directly into source code unless explicitly instructed to read it from the environment instead, which leaks the secret into version control.
deeper: []
related_formulas: []
---

## Why prototypes start with fake data

Ask an AI coding agent or a prototyping tool to build you an app, and it will almost always reach for mocked or in-memory data first — a JavaScript array standing in for a database, state that resets every time the server restarts. This isn't laziness; it's the fastest path to something you can click through and evaluate, and it sidesteps every infrastructure decision until you've confirmed the app's behavior is worth building further. The problem is that this convenience is invisible until the moment it isn't: the prototype looks complete right up until you refresh the page and watch everything you entered disappear.

## Designing a schema instead of letting one happen to you

When you move to a real database, the question "what tables do I need, and how do they relate?" deserves actual thought rather than being answered implicitly by whatever the prototype's mock data happened to look like. A schema designed deliberately — what's required, what's optional, what references what, what should be unique — is far cheaper to get right now than to migrate later, once real user rows depend on the current shape. This is also where you decide what a real database gives you that an in-memory array couldn't: constraints that reject bad data at the source instead of failing somewhere downstream in application code.

## Secrets don't belong in your code

A database connection string is a credential, and credentials that end up committed to a Git repository are compromised the moment that repository is pushed anywhere, public or not. The standard fix is an environment variable — a value injected at runtime, read from a `.env` file locally and from the platform's secret management in production — that your application code references by name but never contains literally. This is a place where directing an AI agent needs an explicit instruction: handed a working connection string in the course of debugging, an agent will often paste it straight into a file to make the immediate problem go away, because that's the most direct path to "it works." Telling it, up front, that secrets go through environment variables and nowhere else heads this off before it happens.

## What breaks when data stops being fake

Mock data is unrealistically well-behaved. It's always present, always the right shape, always fast. Real data is messy — a field a user left blank, a list that's legitimately empty, a query that takes 200 milliseconds instead of returning instantly from memory. Moving to a real database is often the first time these assumptions get tested, and the failures that surface (a crash on an empty array, a page that hangs waiting on a slow query, a form that accepted input the schema now rejects) aren't new bugs so much as pre-existing ones the mock data was hiding. Expect this phase to surface issues, and treat each one as the schema or the code teaching you something about a case you hadn't considered.

## Why a managed platform like Supabase lowers the bar

Running your own Postgres instance means owning backups, patching, and networking on top of the schema work. A managed provider like Supabase gives you a real relational database, built-in authentication, and auto-generated APIs without that operational overhead, which is a large part of why it shows up so often in the path from AI-generated prototype to a shipped product — it covers the gap between "the prototype works" and "the data is real" without asking you to become a database administrator first.
