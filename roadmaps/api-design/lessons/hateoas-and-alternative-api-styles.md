---
title: "HATEOAS and the Richardson Maturity Model"
description: REST's actual hypermedia ideal, why almost no production API reaches it, and SOAP as the contract-first alternative that still survives in enterprise.
roadmap: api-design
node: hateoas-and-alternative-api-styles
group: contract-design
video:
  provider: youtube
  id: "WrO7iGU_nnw"
  start: 0
  end: null
creator:
  name: Milan Jovanović
  channel_url: https://www.youtube.com/@MilanJovanovicTech
duration_seconds: 1118
level: intermediate
prerequisites:
  - pagination-filtering-and-sorting
takeaways:
  - "The Richardson Maturity Model describes four levels of RESTfulness, not four competing styles — Level 0 is one endpoint for everything (RPC-over-HTTP), Level 1 adds resources, Level 2 adds proper HTTP verbs and status codes, and Level 3 adds HATEOAS, hypermedia links that tell the client what it can do next."
  - "Most production APIs — including ones from companies with every resource to do it 'right' — stop at Level 2, and that's a legitimate engineering choice, not a shortcut: Level 2 already gets you resource-oriented URIs, correct verb/status-code semantics, and a predictable contract, which is most of REST's practical value."
  - "HATEOAS's actual pitch is decoupling the client from hardcoded URIs — instead of a client knowing in advance that cancelling an order means `DELETE /orders/482`, the order resource's own response includes a `cancel` link the client follows, so the server can change that URI later without breaking anyone who follows links instead of hardcoding them."
  - "HATEOAS mostly lost because the cost lands asymmetrically — the server does meaningfully more work generating and maintaining correct links, while the client-side benefit (not hardcoding URIs) only pays off for a specific kind of client, a generic hypermedia-following browser-like agent, that almost nobody actually builds; most real clients are written against fixed documentation anyway and gain little from runtime link discovery."
  - "SOAP is contract-first by design — a WSDL document formally specifies every operation, type, and fault before a line of client or server code exists, which trades REST's flexibility for strict, machine-verifiable, tooling-generated contracts that still matter in banking, insurance, and other domains with heavy compliance and legacy-integration requirements."
  - "SOAP's survival in those domains isn't nostalgia — a rigid, schema-validated contract with built-in standards for security (WS-Security) and transactions (WS-AtomicTransaction) is a genuine fit for systems that integrate dozens of vendors over years-long contracts, where REST's looser, documentation-based contract is a liability rather than a flexibility win."
check_questions:
  - q: "What's the actual difference between Level 2 and Level 3 of the Richardson Maturity Model?"
    a: "Level 2 means the API uses proper resource URIs plus correct HTTP verbs and status codes — a solid, predictable REST API by most practical definitions. Level 3 adds HATEOAS on top: responses include hypermedia links telling the client what related actions or resources are available, so the client discovers its next moves from the response itself instead of having that logic hardcoded in advance."
  - q: "Why do most real-world APIs stop at Level 2 instead of reaching Level 3?"
    a: "Because Level 3's benefit — a client that doesn't need to hardcode URIs because it follows links discovered at runtime — mostly pays off for a generic hypermedia-following client that almost nobody builds; most real clients are written against fixed documentation and call known endpoints directly, so the extra server-side cost of generating and maintaining correct links buys little for how APIs are actually consumed in practice."
  - q: "Concretely, what does a HATEOAS response give a client that a plain JSON response doesn't?"
    a: "A plain response for order 482 might just return its fields. A HATEOAS response adds a set of links — for example a `cancel` link pointing at the exact URI and method to cancel that specific order, present only if cancellation is currently a valid action for that order's state — so the client learns what it can legally do next from the data itself, instead of needing separate, out-of-band logic to know that canceling means `DELETE /orders/482` and that doing so is only valid before the order ships."
  - q: "Why does SOAP persist in banking and insurance despite REST dominating almost everywhere else?"
    a: "Those domains integrate with many counterparties over long-lived contracts where a formally specified, machine-verifiable contract (a WSDL) and built-in standards for security and transactional integrity matter more than REST's flexibility and lighter weight — the operational cost of a loosely documented REST contract across dozens of vendor integrations outweighs SOAP's verbosity in exactly those settings."
deeper:
  - label: "HATEOAS & Richardson Maturity Model Explained | REST API Level 3 Deep Dive with Examples"
    url: "https://www.youtube.com/watch?v=UpmezwEPqVI"
related_formulas: []
---

## REST's actual definition, and the gap most APIs live in

"REST" gets used loosely enough in everyday conversation that it's worth going back to what the term originally meant. Roy Fielding's 2000 dissertation, which coined REST, defined it as a set of architectural constraints — one of which is that a client interacts with a system entirely through hypermedia: representations of resources that include the links needed to discover and act on related resources, with no out-of-band knowledge of URIs baked into the client ahead of time. Almost nothing commonly called "a REST API" actually satisfies that constraint. What most of the industry calls REST is closer to "resources addressed by URIs, manipulated with the right HTTP verbs, returning JSON" — which is a real and useful discipline, just not the full thing Fielding specified. The **Richardson Maturity Model**, developed by Leonard Richardson and popularized by Martin Fowler, is the standard way to name that gap.

## The four levels

**Level 0** is the floor: one URI, usually one HTTP method (almost always `POST`), and the actual operation encoded inside the request body — essentially RPC tunneled over HTTP. A single `/api` endpoint that takes `{"action": "getOrder", "id": 482}` is Level 0 regardless of how modern the framework serving it is.

**Level 1** introduces resources: different URIs for different things (`/orders/482` instead of one catchall endpoint), but still typically using one HTTP method for everything and ignoring status code semantics.

**Level 2** adds the two pieces most people mean when they say "REST API" today: correct use of HTTP verbs (`GET` to read, `POST` to create, `PATCH`/`PUT` to update, `DELETE` to remove) and correct use of status codes (`404` for not found, `409` for conflict, `201` for created) to communicate outcomes instead of burying them in the response body. Everything `rest-resource-modeling-and-uri-design` covers — nouns not verbs, collection/item URIs, parameters in the right place — gets an API to Level 2 when combined with proper verb and status-code usage, which `computer-science`'s `dns-and-http` already covers the mechanics of.

**Level 3** is HATEOAS: **H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate. Responses include links describing what the client can do next, and the client follows those links rather than constructing URIs from hardcoded knowledge. A Level 2 response for a shippable order might look like:

```json
{ "id": 482, "status": "pending", "total": 149.00 }
```

The equivalent Level 3 response adds the actions actually available given that order's current state:

```json
{
  "id": 482, "status": "pending", "total": 149.00,
  "links": [
    { "rel": "self", "href": "/orders/482", "method": "GET" },
    { "rel": "cancel", "href": "/orders/482", "method": "DELETE" },
    { "rel": "pay", "href": "/orders/482/payment", "method": "POST" }
  ]
}
```

Once the order ships, a server implementing this correctly stops including the `cancel` link — the client learns "cancellation is no longer valid" from the data itself, rather than needing separate business logic baked in ahead of time to know that shipped orders can't be cancelled.

## Why Level 3 almost never happens, and whether that's a problem

The theoretical appeal is real: a client that only ever follows links never hardcodes a URI, so the server is free to restructure its URI scheme, add new valid actions, or change which actions are available per resource state, all without a client-side deploy. In practice, almost no production API reaches Level 3, and it's worth being honest about why rather than treating it as laziness.

The cost is asymmetric. Generating correct links means the server has to compute, for every response, exactly which actions are valid given the resource's current state — not just "here's an order," but "here's an order, and here is the precise, current, conditionally-available set of things you're allowed to do to it." That's real ongoing engineering work, and it has to stay correct as business rules evolve. The benefit, meanwhile, only accrues to a specific kind of client: one that's actually written to discover and follow links dynamically rather than calling known endpoints it already has documented. Almost nobody builds that kind of client. Mobile apps, frontend SPAs, and server-to-server integrations are virtually always written against a fixed API reference — `openapi-and-api-documentation-tooling`, later in this roadmap, covers how that reference gets produced and kept in sync — and a developer who already knows from the docs that cancelling an order is `DELETE /orders/{id}` gets no practical benefit from also having that URI handed to them inside every response.

So the honest answer to "is skipping Level 3 a problem" is: usually no. A well-documented Level 2 API with a stable, predictable URI scheme gives real clients almost everything HATEOAS promises, at a fraction of the implementation cost. HATEOAS earns its keep in narrower cases — APIs consumed by a genuinely generic client (a workflow engine that's supposed to adapt to new actions without a redeploy, for instance), or state machines complex enough that "which actions are valid right now" is itself valuable information worth shipping in the payload. Outside those cases, treating Level 2 as the practical ceiling is a defensible, common engineering decision, not a corner cut.

## SOAP, briefly, as the contrast

SOAP (Simple Object Access Protocol) predates REST's dominance and represents close to the opposite set of trade-offs. Where REST is resource-oriented and typically documented after the fact (or alongside development, via OpenAPI), SOAP is **contract-first**: a WSDL (Web Services Description Language) document formally specifies every operation, every input and output type, and every possible fault, before any client or server code is written — tooling then generates both ends from that one WSDL, which makes a SOAP contract machine-verifiable in a way a REST API's documentation generally isn't. SOAP messages are XML, always sent over POST regardless of the underlying operation's actual semantics, which is part of why it reads as heavier and less "webby" than REST — `system-design`'s `api-architectural-styles` covers REST vs GraphQL vs gRPC in more depth, and the contrast with SOAP here is the same shape: a trade of flexibility for a stricter, more formally specified contract.

SOAP's continued use in banking, insurance, and other heavily regulated or legacy-integration-heavy domains isn't inertia alone. Those industries integrate with many counterparties over contracts that run for years, where a WSDL's machine-verified contract reduces integration risk across dozens of vendors in a way REST's looser, documentation-based contract doesn't, and where SOAP's built-in extensions — WS-Security for message-level security independent of transport, WS-AtomicTransaction for distributed transactional guarantees — solve problems that a REST API has to solve ad hoc, if at all. It's a genuine fit for that specific environment, not a holdover nobody's gotten around to replacing.
