---
title: "Choosing a Frontend Architecture"
description: SPA, SSR, SSG, and hybrid rendering as an architecture decision — what actually drives the choice, not a framework tutorial.
roadmap: software-architect
node: choosing-a-frontend-architecture
group: data-and-integration-architecture
video:
  provider: youtube
  id: "kUs-fH1k-aM"
  start: 0
  end: null
creator:
  name: Theo - t3.gg
  channel_url: https://www.youtube.com/@theo
duration_seconds: 1094
level: intermediate
prerequisites:
  - microfrontends
takeaways:
  - "A single-page application (SPA) ships a near-empty HTML shell and builds the page client-side with JavaScript — fast subsequent navigations, but a slow or blank-looking first paint, and nothing for a crawler that doesn't execute JavaScript to read."
  - "Server-side rendering (SSR) renders the actual page to HTML on the server for every request, then hands the browser working markup immediately and hydrates it with JavaScript afterward — better first paint and SEO, at the cost of render work (and often a database round trip) on every single request."
  - "Static site generation (SSG) renders every page to HTML once, at build time, and serves the resulting files from a CDN — the fastest possible delivery and the cheapest to operate, but only works when content doesn't need to be different per request."
  - "The real decision driver is how often the content changes and who needs it fast: a marketing page or blog (rarely changes, needs peak SEO and speed) fits SSG; a personalized or frequently updated page (a feed, a dashboard) needs SSR or a client-rendered SPA instead."
  - "Modern frameworks blur these into one hybrid system per route rather than one choice per app — a team can serve a product catalog page via SSG, a user dashboard via SSR, and an interactive editor as a client-rendered island, all in the same codebase."
check_questions:
  - q: "Why does an SPA typically have a slower first paint than SSR, even though it can feel faster once loaded?"
    a: "An SPA's initial HTML response is close to empty — the browser has to download the JavaScript bundle, execute it, and only then render real content — whereas SSR sends a fully rendered HTML page on the very first response, so there's something visible before any client-side JavaScript has even run."
  - q: "What specifically does static site generation trade away to get its speed and low cost?"
    a: "Because every page is rendered once at build time and served as a fixed file, SSG can't show different content to different users or reflect data that changes between deploys — it's a poor fit for anything personalized or frequently updated, which is exactly what SSR or client rendering is for."
  - q: "Given a marketing site and a logged-in user dashboard in the same product, why might an architect choose two different rendering strategies for them rather than one for the whole app?"
    a: "The marketing site's content is the same for every visitor and rarely changes, so SSG gets maximum speed and SEO for free; the dashboard's content is personalized and changes constantly, which SSG structurally can't do, so it needs SSR or client-side rendering instead — picking one strategy for both would either slow the marketing pages down for no reason or make the dashboard impossible to build correctly."
deeper: []
related_formulas: []
---

## The question underneath "which framework"

Teams often frame this as a framework choice — Next.js versus a plain React SPA versus Astro — but the framework is just whichever tool implements the rendering strategy you actually need. The real architectural question is narrower and more durable than any one tool: **where does HTML get generated, and when?** There are three base answers, and a fourth that's really "pick per page."

## Client-side rendering (the SPA)

A single-page application ships the browser a nearly empty HTML document — a `<div id="root">` and a script tag — and does all the real work of building the page in the browser, after the JavaScript has downloaded, parsed, and executed. Once that initial cost is paid, navigating between pages inside the app can feel instant, because the app is just swapping components in memory rather than requesting and rendering a whole new document from the server.

The cost shows up at the edges: the first visit has to wait through a blank or skeleton screen until the JavaScript bundle loads and runs, which hurts both perceived performance and metrics like Largest Contentful Paint. It's also a real problem for SEO and for any crawler or bot that doesn't execute JavaScript — if the page's actual content only exists after a script runs, a crawler reading the raw HTML sees nothing.

## Server-side rendering (SSR)

Server-side rendering answers both of those problems by moving the "build the page" work back onto the server, for every request. When a browser asks for a page, the server runs the same component code that would otherwise run client-side, produces real, fully-formed HTML, and sends that back immediately — the browser has something to paint before a single line of client JavaScript has executed. The page is then **hydrated**: the same JavaScript that rendered on the server attaches its event listeners to the existing markup in the browser, turning static HTML back into an interactive app without re-rendering everything from scratch.

The cost is that this render work — and usually at least one data fetch, like a database query — happens again on every single request. A page viewed by ten thousand people triggers ten thousand server-side renders, even if the underlying content hasn't changed since the last one.

## Static site generation (SSG)

Static site generation sidesteps that repeated cost entirely by doing the render work exactly once, at build time, for every page the site has — producing a folder of plain HTML files that get served straight from a CDN with no per-request rendering at all. This is as fast and as cheap to operate as serving a page can possibly be, because there's no server-side work happening at request time, only a static file being handed back.

The trade-off is inflexibility: because the HTML was generated once, before any particular visitor ever asked for the page, SSG structurally cannot show different content to different users or reflect data that changed since the last build. A product page whose price updates hourly, or a feed that's different for every logged-in user, can't be served this way without either rebuilding constantly or falling back to fetching that one changing piece client-side on top of the static shell.

## The actual decision framework

None of the three is universally "better" — each is the right answer to a different question about the content:

- **How often does this page's content change?** Rarely (marketing pages, documentation, blog posts) favors SSG. Constantly or per-request (a live feed, a search results page, a personalized dashboard) rules SSG out.
- **Does it need to be different per visitor?** Personalized or authenticated content can't be pre-rendered once for everyone, which pushes toward SSR or client-side rendering.
- **How much does first-paint speed and SEO actually matter for this page?** A marketing landing page lives or dies on both; an internal admin tool behind a login wall usually needs neither, which makes a plain SPA's trade-offs perfectly fine there.

Modern meta-frameworks (Next.js, Astro, Remix, SvelteKit) have mostly stopped forcing a single choice for an entire application and instead let a team pick per route: a product catalog page generated statically, a user dashboard server-rendered on every request, and a complex interactive editor shipped as a client-rendered "island" within an otherwise static page. The architect's job isn't picking one winner — it's recognizing that different pages in the same product have genuinely different requirements, and matching the rendering strategy to each one rather than defaulting the whole app to whichever strategy the framework makes the path of least resistance.
