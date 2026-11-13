---
title: From Prototype to Working App
description: The common problems AI-generated prototypes ship with, from fake auth to hardcoded data, and the concrete fixes needed before they're real software.
roadmap: ai-product-builder
node: from-prototype-to-working-app
group: prototyping-with-ai
video:
  provider: youtube
  id: "Llhno97FUmo"
  start: 0
  end: null
creator:
  name: Peter Yang
  channel_url: https://www.youtube.com/@PeterYangYT
duration_seconds: 2357
level: intermediate
prerequisites:
  - prompt-to-app-generation
  - claude-and-chat-based-design-tools
takeaways:
  - "An AI-generated prototype that looks finished usually isn't: hardcoded URLs, fake client-side auth, and baked-in sample data are the default, not the exception."
  - "Client-side-only 'auth' that just hides UI elements isn't security at all — anyone can see the real data by calling the API directly."
  - "Images or files encoded as base64 and stored inline in code or a database column work for a demo and fall over at any real scale."
  - "A missing real schema shows up as data shoved into loosely-typed JSON blobs, which is fine for a prototype and a liability the moment two features need to share that data."
  - "Turning a prototype into a working app is a specific, checklist-shaped job: find each of these shortcuts and replace it with the real mechanism."
check_questions:
  - q: Why is client-side-only auth in a generated prototype not real security?
    a: It typically just conditionally hides or shows UI elements based on a flag in the browser; the underlying API endpoints are still open to anyone who calls them directly, so there's no actual access control protecting the data.
  - q: What's the problem with images or data stored as base64 in a prototype?
    a: Base64-encoding files and embedding them directly in code or database rows is quick to generate and works at demo scale, but it bloats storage and payload size badly and doesn't support real file handling like CDN delivery or independent versioning as usage grows.
  - q: Why does a missing real database schema often go unnoticed until later?
    a: A prototype can fake structured data with loosely-typed JSON blobs that work fine as long as only one feature touches that data; the gap shows up once a second feature needs to query or relate to the same data in a way the blob was never designed for.
deeper: []
related_formulas: []
---

## The gap between looks-done and is-done

An AI-generated prototype can look completely finished — polished UI, working navigation, data flowing across screens — while being built almost entirely out of shortcuts that happen to render correctly in a demo. This isn't a flaw unique to any one tool; it's a predictable consequence of how these tools generate fast, plausible-looking results. They optimize for "it works when I click through it," not for "it survives a second user, a page refresh, or someone reading the network tab." Knowing the specific shortcuts to look for turns an intimidating, vague "make this production-ready" task into a concrete checklist.

## Hardcoded URLs and configuration

The most common shortcut is also the easiest to fix: API endpoints, redirect URLs, and environment-specific values get written directly into the code as literal strings, because that's the fastest way for a generator to produce something that runs immediately. This works fine until you deploy to a second environment, rename a resource, or need the same code to point somewhere different in staging versus production. The fix is mechanical — pull every hardcoded value out into environment variables or a config layer — but it has to be done deliberately, because nothing about the prototype running successfully will tell you it's needed.

## Fake auth is not auth

The more consequential shortcut is authentication that exists only in the browser. A generated app will often show a login screen, store something like `isLoggedIn: true` in local state after a fake check, and use that flag to decide which UI to render. This feels like auth because the experience of using the app matches what real auth would look like. It isn't auth, because nothing stops a request to the underlying API directly, bypassing the UI entirely and reaching data or actions that were supposed to be protected. Fixing this means moving authentication to the server: real session or token verification on every request that touches protected data, not just a conditional render in a component.

## Data that was never meant to last

Two related shortcuts show up constantly: media or files stored as base64 strings embedded directly in code or database columns, and structured data stored as loosely-typed JSON blobs instead of a proper schema. Both are fast to generate because they avoid setting up real infrastructure — a file store, a relational schema with defined relationships — and both work fine for a single demo user clicking through a happy path. Base64-embedded media bloats every payload that touches it and has no path to CDN delivery or caching. JSON-blob data has no enforced shape, so it tolerates inconsistency silently until a second feature tries to query or join against it and discovers the data isn't shaped the way that feature assumed. The fix for both is the same kind of work: move files to real object storage with URLs instead of inline encoding, and design an actual schema with typed columns and relationships before more features get built on top of the blob.

## Treat it as a checklist, not a rewrite

None of this means the prototype was wasted work — it proved out the idea and the interface faster than building from scratch would have. But converting it into something real is a specific, boundable task: walk through auth, data storage, file handling, and configuration one at a time, and replace each shortcut with its real-world equivalent. Doing this early, before more features get layered on top of the fake foundations, is far cheaper than discovering the gaps after a real user hits one.
