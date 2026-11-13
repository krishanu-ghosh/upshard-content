---
title: "Pagination, Filtering, and Sorting"
description: Offset versus cursor pagination and why cursors win at scale, plus the query-parameter and response-envelope conventions that make large collections usable.
roadmap: api-design
node: pagination-filtering-and-sorting
group: contract-design
video:
  provider: youtube
  id: "mvlzhBgGS4s"
  start: 0
  end: null
creator:
  name: Caleb Curry
  channel_url: https://www.youtube.com/@codebreakthrough
duration_seconds: 1546
level: intermediate
prerequisites:
  - api-versioning-strategies
takeaways:
  - "Offset pagination (`?offset=10000&limit=20`) is simple and lets a client jump to an arbitrary page, but the database still has to walk and discard every row before the offset, so cost grows linearly with page depth — and it's not a cosmetic slowdown, it's a query that gets measurably worse the deeper a client pages."
  - "Offset pagination also silently corrupts results under concurrent writes: if a row is inserted before the current offset while a client is paging, every subsequent page shifts by one, producing skipped or duplicated items with no error to signal it happened."
  - "Cursor pagination replaces a numeric offset with an opaque pointer to the last item seen — usually an encoded, indexed, unique column like `id` or `created_at` — so each page is fetched with a `WHERE id > :cursor ORDER BY id LIMIT :n` that costs the same regardless of depth and is stable under concurrent inserts."
  - "Cursor pagination's real cost is giving up random access — you can't jump to \"page 50\" with a cursor, only walk forward (and, if you encode it both ways, backward) from a known point, which is the right trade for infinite scroll and large exports and the wrong one for a UI with numbered page links."
  - "Filtering and sorting both live in query parameters on the collection endpoint, never as separate paths — `GET /orders?status=shipped&sort=-created_at` keeps `/orders` as the one resource and lets the parameters compose freely, instead of needing a new endpoint for every combination a client might want."
  - "A paginated response needs an envelope, not a bare array, so pagination metadata (the next cursor, whether more pages exist, optionally a total count) has somewhere to live without overloading response headers or inventing a different shape per endpoint."
check_questions:
  - q: Why does offset pagination get slower the deeper a client pages, even though the page size stays constant?
    a: The database still has to scan past every row before the requested offset to know where the page starts, even though it only returns `limit` rows — `OFFSET 50000 LIMIT 20` costs roughly the work of scanning 50,020 rows, not 20, so cost scales with how deep into the collection the client has paged, not with the page size.
  - q: A client is paging through a list with offset pagination while new items are being inserted at the front. What goes wrong?
    a: Every insert ahead of the current offset shifts the position of every row after it by one, so the next page the client requests is now misaligned with what it already saw — it will either see an item twice (if growth pushed a previously-seen item back into view) or miss one entirely (if growth pushed an unseen item past the current page), and nothing in the response indicates this happened.
  - q: What has to be true about the column a cursor is built on for cursor pagination to work correctly?
    a: It needs to be indexed (so the `WHERE col > :cursor` lookup is fast regardless of depth), unique or made unique by combining it with a tiebreaker like the primary key (so no two rows produce the same cursor and get silently skipped or duplicated), and stable (its value shouldn't change after the row is created, or a client's cursor could point at a moving target).
  - q: Why can't a UI that needs numbered page links ("1 2 3 ... 10") use pure cursor pagination?
    a: Cursor pagination only knows how to say "give me the page after this specific item," not "give me page 7 directly" — there's no way to compute an arbitrary cursor without having already walked to that position, so a UI that needs to jump straight to an arbitrary page number needs offset pagination (or a hybrid), accepting its consistency trade-offs, instead.
deeper:
  - label: "Web API Pagination | Offset-based vs Cursor-based"
    url: "https://www.youtube.com/watch?v=WUICbOOtAic"
related_formulas: []
---

## Why "just return everything" stops working

A collection endpoint that returns every row works fine in a demo and breaks the moment the underlying table has real volume — the response grows unbounded, serialization time grows with it, and a client that only needed the first twenty results pays the full cost anyway. Pagination exists to cap the size of a single response and let a client fetch a large collection incrementally instead of all at once. The design question isn't whether to paginate a collection endpoint — any endpoint that can return an unbounded number of rows should — it's which pagination style to use, and the two real options behave very differently once the underlying data gets large.

## Offset pagination and why it degrades

Offset pagination is the version everyone reaches for first because it matches how people already think about pages: `GET /orders?offset=40&limit=20` means "skip the first 40, give me the next 20." It's trivial to implement on top of SQL's `OFFSET`/`LIMIT`, and it lets a client jump straight to an arbitrary page — "show me page 7" is a direct, stateless request.

The cost hides in how the database actually executes that query. `OFFSET 50000 LIMIT 20` doesn't mean "find row 50,000 directly" — the database has to walk and discard the 50,000 rows ahead of the offset before it can hand back the 20 you asked for, because relational indexes are built to find a row by key, not by its ordinal position in a result set. The deeper a client pages, the more work every subsequent request costs, even though the page size never changes. On a table with a few thousand rows this is invisible. On a table with tens of millions of rows and a client paging to page 500, it's a measurably slow query — and it's slow in a way that gets worse continuously as the table grows, which is exactly the kind of degradation that doesn't show up in a staging environment with seed data.

The second problem is correctness, not just speed, and it's easy to miss because it fails silently. Offset pagination identifies a page by position, not by content. If a row is inserted ahead of the client's current offset between two of its requests — entirely plausible on any table that takes writes while being read — every row after that insertion shifts by one position. The client's next "page 8" request now returns a result set that's offset by one from what it would have seen without the write: either it re-sees an item it already had, or it skips one it never saw. Nothing in the response signals this happened. For a product catalog browsed occasionally, this is a minor annoyance. For an export job, an audit log, or any feed where completeness matters, it's a correctness bug baked into the pagination mechanism itself.

## Cursor pagination and why it scales

Cursor pagination (also called keyset pagination) replaces "skip N rows" with "give me rows after this specific point." Instead of an offset, the client sends an opaque cursor — in practice, an encoded value derived from the last item it saw, usually a unique, indexed, monotonic column like `id` or `created_at`. The query becomes `WHERE id > :cursor ORDER BY id LIMIT :n`, which the database can execute directly against the index without scanning anything before the cursor position — the cost of fetching page 500 is the same as fetching page 1, because there is no "page 500" as a concept the database has to compute; there's only "the next 20 rows after this specific id."

This also fixes the concurrency problem, because a cursor identifies content, not position. An insertion anywhere in the table doesn't shift what "after id 48213" means — that boundary is still exactly where it was. A client paging forward through a cursor-based feed sees a consistent walk through the data even while the underlying table is being written to, which is why cursor pagination is the default for feeds, activity logs, and any API serving a dataset that changes while clients are actively reading it.

The trade is giving up random access. A cursor only knows how to go "forward from here" (and "backward from here," if you maintain cursors in both directions) — there's no way to compute the cursor for an arbitrary page number without having already walked to it, so a UI that needs literal numbered page links genuinely cannot be built on pure cursor pagination. That's not a flaw to work around, it's the actual trade-off: pick offset pagination when a client needs to jump to an arbitrary page and the collection is small or slow-changing enough that the degradation doesn't matter; pick cursor pagination when the collection is large, changes concurrently, or is consumed as an infinite-scroll or streaming-style feed rather than numbered pages.

## Filtering and sorting live beside pagination, not instead of it

Filtering and sorting answer a different question than pagination — not "how much of the collection to return," but "which subset, in what order" — and both belong as query parameters on the same collection endpoint, composing freely with pagination rather than requiring their own routes. `GET /orders?status=shipped&sort=-created_at&limit=20` filters to shipped orders, sorts newest-first, and caps the page size, all as one request against the one `/orders` resource — this is the query-parameter role described in `rest-resource-modeling-and-uri-design`. A leading `-` on a sort field is a common, readable convention for descending order (`sort=-created_at` vs `sort=created_at`); supporting a comma-separated list (`sort=-priority,created_at`) covers multi-field sorts without inventing new syntax. Filters should map onto real, documented fields rather than accepting arbitrary expressions — `?status=shipped` is safe and indexable; letting a client pass a raw query fragment is both a performance risk and, done carelessly, an injection risk.

## The envelope a paginated response needs

A bare JSON array works for an unpaginated response, but a paginated one needs somewhere to put metadata the client needs to fetch the next page — at minimum the cursor (or offset) for the next request and whether more results exist. The common shape wraps the items in an envelope object:

```json
{
  "data": [ { "id": 482, "status": "shipped" }, ... ],
  "pagination": {
    "next_cursor": "eyJpZCI6NDgyfQ==",
    "has_more": true
  }
}
```

A total count is sometimes included but should be treated as optional and possibly expensive — computing an exact `COUNT(*)` on a filtered query can cost as much as the query itself on a large table, so some APIs omit it, approximate it, or make it an opt-in field rather than paying for it on every request by default. Whatever shape you choose, keep it identical across every paginated endpoint in the API — a client that has to remember a different envelope per resource has effectively lost the benefit of a consistent contract, which is the same discipline `structured-error-handling-and-rfc-7807`, later in this group, applies to error responses.
