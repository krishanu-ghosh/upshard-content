---
title: "REST Resource Modeling and URI Design"
description: How to think in resources instead of actions, and the concrete rules that turn that thinking into clean, predictable URIs.
roadmap: api-design
node: rest-resource-modeling-and-uri-design
group: contract-design
video:
  provider: youtube
  id: "36--CQc19u4"
  start: 0
  end: null
creator:
  name: Milan Jovanović
  channel_url: https://www.youtube.com/@MilanJovanovicTech
duration_seconds: 780
level: beginner
prerequisites: []
takeaways:
  - "A URI names a thing, never an action — the HTTP method already carries the verb, so `POST /orders` is correct and `POST /createOrder` is a tell that the API is RPC wearing REST's clothes."
  - "Collections are plural nouns (`/orders`), a single resource inside one is that same plural noun plus an identifier (`/orders/482`) — the pattern never switches to singular just because you're addressing one item."
  - "Nesting should mirror genuine ownership, not every relationship in the domain — `/customers/42/orders` is right because an order belongs to exactly one customer, but a deeply nested path like `/customers/42/orders/482/line-items/3/product` is a sign to flatten: give line items and products their own top-level collections and let the response carry the links."
  - "Path segments identify resources; query parameters filter, sort, and paginate within a resource; the request body carries the representation being created or changed — mixing these up (an `?id=` query param instead of a path segment, a resource name stuffed into a query string) is the most common amateur signal in a REST API."
  - "Casing and pluralization conventions exist to be boring and consistent, not clever — lowercase, kebab-case for multi-word segments (`/line-items`, not `/lineItems` or `/line_items`), and a project-wide pluralization rule applied without exceptions."
check_questions:
  - q: Why is `GET /users/getActiveUsers` a worse design than `GET /users?status=active`?
    a: The first bakes a verb and a specific filter into the URI itself, which means every new filtering need requires a new endpoint; the second treats `/users` as the one resource collection and uses a query parameter to narrow it, so the same endpoint scales to any filter without multiplying URIs.
  - q: When does nesting a resource under its parent stop being the right call?
    a: When the child doesn't belong exclusively to one parent, or when the nesting goes more than one or two levels deep — at that point flatten the child into its own top-level collection and connect it to the parent through an identifier field or a link in the response body, rather than encoding the whole ownership chain into the path.
  - q: A client wants to create a new order. Where does the order's data go — the URI or the request body?
    a: The request body. The URI (`POST /orders`) names the collection the new resource will join; the body carries the representation of the thing being created. Nothing about the order's contents belongs in the path or query string.
  - q: Why does consistent pluralization matter more than which convention you pick?
    a: Because the cost isn't singular-vs-plural, it's a client having to special-case every endpoint to remember which one it was — a project that's rigorously plural everywhere is more usable than one that's "correctly" singular for singleton resources but inconsistent about applying that rule.
deeper:
  - label: "REST API Series | Tutorial 6: How to design URI/URL?"
    url: "https://www.youtube.com/watch?v=SBeG6cl8BlA"
related_formulas: []
---

## Nouns, not verbs

The single biggest mental shift in REST resource modeling is this: a URI identifies a *thing*, and the HTTP method says what you're doing to it. Once that clicks, most naming questions answer themselves. `GET /invoices/9091` reads invoice 9091. `DELETE /invoices/9091` deletes it. You never need `GET /getInvoice?id=9091` or `POST /deleteInvoice`, because the verb already lives in the method — `system-design`'s `api-architectural-styles` covers why REST chose this shape over RPC's "call a named function" model in the first place, and `computer-science`'s `dns-and-http` covers what GET, POST, PUT, PATCH, and DELETE actually do on the wire. This lesson assumes you already have both; it's about what happens once you've committed to REST and have to design the resource graph itself.

The practical test for "is this a resource" is whether it survives being addressed by an identifier and manipulated with a handful of generic verbs. "Order," "customer," "invoice line item" pass easily. "Send a welcome email" does not — there's no persistent thing being created, read, updated, or deleted, just an action with a side effect. Actions like that are the honest exception to resource modeling, and the cleanest way to handle them is to model the *result* as a resource: `POST /password-reset-requests` creates a password reset request (and triggers the email as a side effect), rather than inventing `POST /sendPasswordResetEmail`. It's a small reframe, but it keeps the URI vocabulary limited to nouns even at the edges.

## Collections, items, and the identifier boundary

A collection is a plural noun: `/articles`, `/subscriptions`, `/api-keys`. An item inside that collection is the same plural noun with an identifier appended: `/articles/4471`. This pattern should never flex — don't switch to a singular path segment just because you're addressing one resource (`/article/4471` is a different, inconsistent vocabulary from `/articles`), and don't invent a second URI shape for "the one you're currently working with" versus "the collection." If a resource is a true singleton — there's exactly one of it per context, like "the authenticated user's profile" — `/me` or `/account/profile` is fine precisely because there's no ambiguity about which one you mean; that's the rare legitimate exception to pluralization, not a loophole for anything you'd rather not pluralize.

Identifiers themselves deserve a decision up front: sequential integers are the simplest and most debuggable but leak information (order volume, creation order) and invite enumeration attacks if authorization isn't airtight; UUIDs or another opaque token avoid both at the cost of a less readable URI. Pick one scheme per resource type and don't mix them within a collection — a client parsing `/orders/{id}` should never have to handle both `"482"` and `"a1b2c3d4-..."` showing up in the same field.

## How deep should nesting go

Nesting a resource under its parent — `/customers/42/orders` — is the right call exactly when the child is meaningless without the parent and belongs to exactly one of them. An order genuinely belongs to one customer; that's real ownership, and the nested path documents it for free. The mistake is extending that same instinct indefinitely: `/customers/42/orders/482/line-items/3/product` tries to encode an entire object graph into one path, and it breaks the moment any single link in that chain needs to be addressed on its own (you can't fetch "that product" without re-walking the whole chain, and you can't change which customer owns order 482 without the URI itself implying a different resource).

The fix is to flatten past one level of real ownership and let the response body carry the rest as links or embedded identifiers: `/orders/482/line-items` is fine (line items genuinely belong to exactly that order), but line items reference a `product_id` that points at a top-level `/products/{id}` collection rather than nesting products under line items under orders under customers. This is also where `hateoas-and-alternative-api-styles` picks up the thread — hypermedia links in the response are the mechanism for telling a client "here's how to get the related resource" without the URI having to spell out the whole path to it.

## Where parameters actually belong

Three places exist for data in a request, and REST assigns each a specific job. **Path segments** identify which resource you mean — they're required, and changing one changes what you're addressing (`/orders/482` and `/orders/483` are different things, not the same thing with a different filter). **Query parameters** narrow or shape a request against a resource that's already been identified — filtering (`?status=shipped`), sorting (`?sort=-created_at`), and pagination (`?cursor=...`) all belong here, and `pagination-filtering-and-sorting`, the next lesson in this group, covers the conventions for that vocabulary in depth. **The request body** carries the representation being sent — the full or partial state of a resource on a `POST`, `PUT`, or `PATCH`.

The error that gives away an inexperienced API is mixing these up: an `?orderId=482` query parameter instead of `/orders/482` in the path, or a resource's identity encoded only in the request body of a `GET` (which many HTTP clients and intermediaries won't even reliably forward). If a piece of data identifies *which* resource, it's a path segment. If it shapes *which subset or order* of an already-identified collection, it's a query parameter. Everything else is the body.

## A worked example

Consider a blogging API. The naive version might offer `/getPublishedPosts`, `/post/create`, `/editPost?id=9`, and `/comment/add?postId=9`. Reworked with resource modeling: `GET /posts?status=published` lists published posts (a noun, filtered by query param, not a bespoke endpoint); `POST /posts` creates one (verb from the method, data in the body); `PATCH /posts/9` edits it (the identifier is a path segment, not a query param); and `POST /posts/9/comments` adds a comment, nested one level because a comment genuinely and permanently belongs to the post it's on. Every one of those four endpoints follows directly from "nouns for resources, methods for verbs, path segments for identity, query params for shaping, body for representation" — the whole discipline is really just refusing to improvise once those four rules are fixed.
