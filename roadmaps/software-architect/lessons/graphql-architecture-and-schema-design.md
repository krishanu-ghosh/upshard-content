---
title: "GraphQL Architecture & Schema Design"
description: What a GraphQL schema actually buys an architecture over REST or gRPC, and the N+1 trap every resolver-based API eventually hits.
roadmap: software-architect
node: graphql-architecture-and-schema-design
group: data-and-integration-architecture
video:
  provider: youtube
  id: "zVNrqo9XGOs"
  start: 0
  end: null
creator:
  name: Apollo GraphQL
  channel_url: https://www.youtube.com/@ApolloGraphQL
duration_seconds: 2398
level: intermediate
prerequisites:
  - etl-and-data-warehousing
takeaways:
  - "GraphQL replaces many fixed REST endpoints with one endpoint and a typed schema — the client specifies exactly which fields it wants in one request, instead of over-fetching a whole resource or under-fetching and needing a second round trip."
  - "The schema is a contract, not a database model — it should be designed around what clients need to ask, with its own types and resolvers, rather than mirroring the shape of the underlying tables or services one-to-one."
  - "The N+1 problem is GraphQL's signature trap — a list query returning a hundred items, each independently triggering its own resolver call to fetch a related field, turns one logical request into a hundred-and-one database round trips."
  - "A batching layer like DataLoader fixes N+1 by collecting all the individual key lookups a resolver layer makes within one tick of the event loop and issuing them as a single batched query, then handing each caller back its own slice of the result."
  - "GraphQL is not a universal replacement for REST or gRPC — it earns its complexity where clients have varied, evolving data needs (a mobile app and a web app wanting different slices of the same graph); for a narrow, stable internal service-to-service call, REST or gRPC's simplicity usually wins."
check_questions:
  - q: "What problem does a single GraphQL endpoint with a typed schema solve that many REST endpoints don't?"
    a: "It lets each client ask for exactly the fields it needs in one request, which avoids over-fetching (REST returning a whole resource when only two fields were needed) and under-fetching (REST requiring a second request to fetch a related resource)."
  - q: "Concretely, how does the N+1 problem happen in a GraphQL resolver?"
    a: "A query resolves a list of N items, and then, for a field on each item that comes from a different data source, GraphQL calls that field's resolver once per item — so one list fetch plus N per-item fetches becomes N+1 total round trips to the database or service."
  - q: "How does DataLoader avoid turning N+1 into N+1 anyway?"
    a: "Instead of each resolver call hitting the database immediately, DataLoader collects the keys requested during the same tick of the event loop, then issues one batched query for all of them at once and distributes the matching result back to each original caller — turning N calls into one."
deeper:
  - label: "GraphQL Explained in 100 Seconds (Fireship)"
    url: "https://www.youtube.com/watch?v=eIQh02xuVw4"
related_formulas: []
---

## One endpoint, a typed graph, and a query the client writes

`system-design`'s `api-architectural-styles` and `rpc-and-grpc` already cover REST's resource-per-endpoint model and gRPC's contract-first binary RPC. GraphQL is a third shape, and it solves a specific pain both of those can have: REST endpoints are designed in advance around what the server thinks a client needs, so a client that needs a slightly different slice of data either over-fetches (gets a whole `User` object back to read one field) or under-fetches (has to make a second request for the user's orders, because the first endpoint didn't include them).

GraphQL inverts that: there's one endpoint, backed by a **schema** — a typed description of every object, field, and relationship the API exposes — and the client sends a **query** describing exactly the shape of data it wants back. Ask for a user's `name` and their `orders { total, createdAt }` in one request, and that's exactly what comes back: no more, no less, in one round trip. Each field in the schema is backed by a **resolver** — a function that knows how to fetch that field's value, whether from a database, a cache, or another service entirely.

## Designing a schema as a contract, not a database mirror

The schema is the whole API's public contract, and the most common mistake with a new GraphQL API is building the schema by copying the database's tables one-for-one. That tends to leak internal implementation detail into the contract — column names, foreign-key shapes, whatever normalization the operational database happens to use (see `etl-and-data-warehousing` for why that shape exists in the first place) — and it locks client code to details that have nothing to do with what clients actually need to ask.

A better schema is designed from the client's point of view: what questions does a screen need answered, and what's the most natural shape for that answer, regardless of how many tables or services it takes to assemble it behind the scenes? A `Product` type in the schema might stitch together a row from the catalog database, a price from a pricing service, and a review score computed on the fly — the client never needs to know that, and the schema's job is to hide it. This is also what makes a schema evolvable: you can change how a field is resolved internally without ever touching the client-facing contract, as long as the field's type and meaning stay the same.

## The N+1 problem — GraphQL's signature performance trap

Resolvers are what make GraphQL's flexibility possible, and they're also where its most famous performance bug lives. Say a query asks for a list of 100 blog posts, and for each post, its author's name. GraphQL resolves the list first (one query), then — because each post's `author` field has its own resolver — calls that resolver once per post. That's 100 more queries, each fetching one author by id, for 101 total round trips to answer one logical question. This is the **N+1 problem**: one query for the list (the "1"), plus N queries for a related field on each item in that list.

It's easy to miss in development, where the list has three items and nobody notices three extra queries. It becomes a real production incident when the list has thousands of items and the database connection pool runs out.

## DataLoader and batching — fixing N+1 without changing the schema

The standard fix doesn't touch the schema or the query the client sent — it changes how resolvers fetch data. A library like **DataLoader** sits between a resolver and the database: instead of each of the 100 `author` resolver calls hitting the database immediately, DataLoader collects every key requested during the same tick of the event loop — all 100 author ids — and then issues **one** batched query (`WHERE id IN (...)`) for all of them at once. Each of the 100 original callers gets its own result back from that single batch, as if nothing changed from its point of view. The resolver code barely changes; the number of round trips drops from 101 to 2.

## When GraphQL is, and isn't, the right call

GraphQL earns its added complexity — a schema to maintain, a resolver layer, a batching strategy — where clients genuinely have varied and evolving data needs: a mobile app and a web dashboard both reading from the same backend but wanting very different slices of it, or a product surface that changes its data needs every sprint. For a narrow, stable, service-to-service call where both sides agree on exactly what's being asked for every time, REST's simplicity or gRPC's performance usually wins outright — introducing a schema and a resolver graph there is solving a problem that call never had.
