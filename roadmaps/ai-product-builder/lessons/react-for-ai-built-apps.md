---
title: React for AI-Built Apps
description: Core React concepts — components, props, state, lists, and events — the shape most AI app generators output for you to read.
roadmap: ai-product-builder
node: react-for-ai-built-apps
group: core-skills-you-still-need
video:
  provider: youtube
  id: "SqcY0GlETPk"
  start: 0
  end: null
creator:
  name: Programming with Mosh
  channel_url: https://www.youtube.com/@programmingwithmosh
duration_seconds: 4804
level: beginner
prerequisites:
  - html-css-and-javascript-fundamentals
takeaways:
  - A React component is a function that returns markup, and a page is just components nested inside other components, each owning a piece of the overall UI.
  - Props pass data from a parent component into a child, one-directionally; a child can't reach back up and change a prop it was given.
  - State is data a component remembers between renders — when state changes, React re-runs the component function and updates the page to match.
  - Rendering a list in React almost always means mapping an array of data to an array of elements, and each item needs a stable key so React can track it correctly.
  - Most AI-generated prototypes emit plain JSX without TypeScript's type annotations, so missing types in code you're reading is normal, not a sign something's broken.
check_questions:
  - q: What's the relationship between props and state in a React component?
    a: Props are data handed down from a parent component and read-only from the child's point of view, while state is data a component owns and manages itself, which can change over time and trigger a re-render when it does.
  - q: Why does a list rendered in React need a "key" on each item?
    a: React uses the key to match items between renders so it can tell which ones were added, removed, or reordered; without a stable key, React may update the wrong DOM elements when the underlying array changes.
  - q: If you open AI-generated React code and see no type annotations anywhere, does that mean something's wrong?
    a: No — it likely means the code is plain JSX rather than TypeScript, which is actually the more common output of AI app generators; the component, props, and state concepts work identically either way, just without compile-time type checking.
deeper: []
related_formulas: []
---

## Components as the unit of everything

React's central idea is that a user interface is built out of components, and a component is just a function that returns a description of what should appear on screen. A button, a login form, an entire dashboard page — each can be a component, and components nest inside each other the way HTML elements nest inside each other, except now each nested piece can carry its own logic. When you open a file an AI tool generated for a React app, you're almost always looking at one of these functions: some JSX-looking markup near the bottom, and some setup — variables, other function calls — above it that determines what that markup ends up showing.

## Props: data flowing one way, down

A parent component hands data to a child component through props, short for properties. If a `UserCard` component needs a name and an avatar URL to render, the component that places `UserCard` on the page passes those in, and `UserCard` reads them as if they were function arguments — because that's exactly what they are. The flow is strictly one-directional: a child can use a prop, but it cannot reach back up and change the value the parent gave it. When something needs to change in response to user action, the convention is for the parent to pass a function down as a prop, and the child calls that function to ask the parent to make the change. Spotting this pattern — a prop that's a function, called from deep inside a child — is one of the most useful things you can learn to recognize in generated code.

## State: what a component remembers

Props come from outside; state lives inside. State is data a component tracks itself and that can change over its lifetime — whether a modal is open, what text is currently in an input, how many items are in a cart. In modern React this is almost always declared with the `useState` hook, which returns the current value and a function to update it. The detail worth internalizing is that calling that update function doesn't just change a variable quietly — it tells React to re-run the component function and redraw whatever depends on that value. Bugs where "the screen isn't updating" are frequently really "state was changed in a way React didn't notice," which is a specific, identifiable category once you know to look for it.

## Lists and conditions: the two recurring patterns

Two shapes cover a large fraction of real UI code. The first is rendering a list: given an array of data — products, messages, todo items — a component typically calls `.map()` to turn each entry into a piece of markup, producing an array of elements React can display. Each of those elements needs a `key` prop, a stable identifier, so React can track which item is which across updates rather than guessing from position. The second shape is conditional rendering: showing one thing or another (or nothing) depending on some piece of state or a prop, usually written with a ternary or a short-circuited `&&`. Once you can spot both patterns on sight, a surprising amount of unfamiliar React code stops looking unfamiliar.

## A note on TypeScript versus plain JSX

The video behind this lesson teaches React with TypeScript, which adds type annotations — things like `name: string` next to a prop — so mistakes get caught before the code even runs. Most AI prototyping tools, including the ones you've likely already tried, output plain JSX instead: the same components, props, and state, with no type annotations at all. If the code in front of you has none of the colons-and-types syntax from the video, that's not a stripped-down or broken version of what you watched — it's simply the more common output format, and every concept you just learned transfers directly.
