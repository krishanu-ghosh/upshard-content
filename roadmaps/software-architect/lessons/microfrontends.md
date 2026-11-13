---
title: "Microfrontends"
description: Splitting a frontend into independently built and deployed pieces — what it buys a large team, and what it costs at runtime.
roadmap: software-architect
node: microfrontends
group: data-and-integration-architecture
video:
  provider: youtube
  id: "fkgQtEAZw4M"
  start: 0
  end: null
creator:
  name: ByteMonk
  channel_url: https://www.youtube.com/@ByteMonk
duration_seconds: 640
level: intermediate
prerequisites:
  - graphql-architecture-and-schema-design
takeaways:
  - "A microfrontend architecture splits a single-page application into several independently built, tested, and deployed pieces, each usually owned end to end by one team, then composes them into one experience at build time or runtime."
  - "The organizational motivation mirrors microservices on the backend (see system-design's monolith-vs-microservices): large frontend monoliths create the same release-train coupling problem, where every team's change has to wait for the same build and deploy."
  - "Module Federation (a Webpack/Rspack capability) is the most common modern mechanism — it lets one already-running application dynamically load another team's already-built JavaScript bundle over the network at runtime, rather than at the host's own build time."
  - "Composition can happen at build time (stitching bundles together before deploying), at runtime in the browser (Module Federation, iframes, web components), or at the edge/server (server-side includes) — each trades off coupling, performance, and operational complexity differently."
  - "The costs are real and easy to underestimate: duplicated dependencies bloating the shipped bundle unless shared carefully, inconsistent UI unless a shared design system is enforced, and cross-fragment state management that has no single obvious owner the way a single SPA's state does."
check_questions:
  - q: "What organizational problem do microfrontends solve that is the frontend equivalent of a backend monolith's problem?"
    a: "A single large frontend codebase forces every team to coordinate on the same build, test, and deploy pipeline, so one team's slow or broken change can block everyone else's release — the same release-train coupling that motivates splitting a backend monolith into microservices."
  - q: "What does Module Federation actually let one application do that it couldn't do before?"
    a: "It lets an already-running application load another team's independently built and deployed JavaScript bundle directly over the network at runtime, rather than that code needing to be included in the host's own build — so each team can ship its piece on its own schedule."
  - q: What's one real cost of a microfrontend architecture beyond the extra operational complexity of running more deployables?
    a: "Without deliberate dependency sharing, each independently built fragment can bundle its own copy of common libraries like React, bloating the total JavaScript shipped to the browser — and without a shared design system, fragments built by different teams tend to visually drift apart."
deeper:
  - label: "Micro-frontends: A microservice approach to the modern web (NearForm, full conference talk)"
    url: "https://www.youtube.com/watch?v=ftBc8w-lwmY"
related_formulas: []
---

## The same split, one layer up the stack

`system-design`'s `monolith-vs-microservices` lesson covers why and when a backend gets split into independently deployable services: team contention over one release train, the need for independent scaling, and the organizational signal that multiple teams keep blocking on each other's changes. A **microfrontend architecture** is the same idea applied to the frontend — instead of one large single-page application that every team contributes to and that ships as one build, the UI is split into several independently built, tested, and deployed fragments, each usually owned end to end by one team, then assembled into a single experience for the end user.

The trigger is almost always organizational before it's technical: a checkout team, a search team, and a recommendations team all shipping into the same frontend repository eventually means nobody can deploy without coordinating with the other two, and a bug in one team's code can block or break a release for all three. Splitting the frontend along those same team boundaries lets each team build, test, and ship its piece on its own schedule — the same payoff a backend split buys, now applied to the UI.

## How composition actually happens

Having independently built fragments only solves half the problem — something still has to stitch them into one page a user experiences as a single, coherent application. There are three broad strategies:

- **Build-time composition.** Each team publishes its fragment as a package; a build step pulls the latest versions and bundles them together before deploy. Simple to reason about, but it reintroduces some of the coordination cost microfrontends were meant to remove — every team's new version still has to go through one shared build.
- **Runtime composition in the browser.** The host application loads each fragment's code directly in the browser, independently of its own build. **Module Federation** (originally a Webpack 5 feature, now also available in Rspack and Vite) is the dominant modern mechanism: a "remote" application exposes specific modules, and a "host" application dynamically imports them over the network at runtime, as if they'd always been part of its own bundle — except that remote's code was built and deployed completely independently. Older variants of runtime composition used iframes (strong isolation, weaker integration) or web components (each fragment registers a custom element the host simply drops into the page).
- **Edge or server-side composition.** Fragments are assembled into one HTML response before it ever reaches the browser, using techniques like server-side includes — composition happens once, server-side, rather than being re-assembled in every visitor's browser.

Each option trades isolation, performance, and operational complexity differently: build-time composition is the simplest to reason about but the most coupled; runtime browser composition is the most independent but pushes the assembly cost (and the risk of version mismatches between fragments) into every user's browser.

## What it actually costs

None of this is free, and the costs mirror backend microservices' costs closely:

- **Duplicated dependencies.** If three independently built fragments each bundle their own copy of React, a user's browser downloads React three times unless the architecture deliberately shares common dependencies between host and remotes — a problem a single SPA's one build step never has.
- **Design drift.** Fragments built by different teams on different schedules tend to visually and behaviorally diverge unless a shared design system (shared components, shared tokens) is enforced across all of them — the same discipline this site's own `/styleguide` exists to enforce for Upshard.
- **Cross-fragment state.** A single SPA has one obvious place to put application state. Once the UI is split into independently owned fragments, something has to own state that spans more than one fragment — a shopping cart that the product-listing fragment and the checkout fragment both need to see — and there's no single default answer for where that state lives.

## When the split is worth it

Microfrontends earn their complexity at genuine organizational scale — multiple teams, each needing true release independence on one shared UI surface — the same bar `monolith-vs-microservices` sets for the backend. For a product built by one team, or even several teams who can coordinate a shared release cadence without real pain, a single frontend codebase with good internal module boundaries gets most of the maintainability benefit without paying the runtime composition tax at all.
