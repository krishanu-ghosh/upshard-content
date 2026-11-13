---
title: "CORS and Content Negotiation"
description: Preflight mechanics and the Access-Control headers that make cross-origin requests work, plus how the Accept header negotiates response shape and version.
roadmap: api-design
node: cors-and-content-negotiation
group: contract-design
video:
  provider: youtube
  id: "tcLW5d0KAYE"
  start: 0
  end: null
creator:
  name: Akshay Saini
  channel_url: https://www.youtube.com/@akshaymarch7
duration_seconds: 755
level: intermediate
prerequisites:
  - rest-resource-modeling-and-uri-design
takeaways:
  - "CORS is a browser-enforced restriction, not a server-side or network-level one — the request still reaches the server and the server still responds; the browser is what inspects the response's `Access-Control-*` headers and decides whether to let the calling JavaScript actually read the result."
  - "A preflight — an automatic `OPTIONS` request the browser sends before the real one — only fires for requests the spec calls 'non-simple': a method other than GET/HEAD/POST, a `Content-Type` other than form-encoded/multipart/plain-text (so `application/json` triggers it), or custom headers like `Authorization`; a plain `GET` with no custom headers never triggers one."
  - "`Access-Control-Allow-Origin` is the one header that actually grants access — it must echo the exact requesting origin (or `*` for fully public, credential-free endpoints), and a mismatch here is the single most common cause of a CORS error regardless of what else is configured correctly."
  - "Credentialed requests (cookies, HTTP auth) require the server to echo back the exact origin — never `*` — in `Access-Control-Allow-Origin`, and to send `Access-Control-Allow-Credentials: true`; a wildcard origin is specifically disallowed for credentialed requests by the spec, as a deliberate guard against one compromised client exposing every user's cookies to every origin at once."
  - "Content negotiation is the `Accept`/`Content-Type` mechanism for a client and server to agree on representation — `Accept: application/json` versus `Accept: application/xml` is the simple case, but the same mechanism extends to media-type versioning (`Accept: application/vnd.myapi.v2+json`), which is the content-negotiation alternative to URI versioning covered in `api-versioning-strategies`."
  - "`Content-Type` on a request describes what the client is sending; `Accept` on a request describes what the client is willing to receive back — conflating the two (or omitting `Accept` and assuming a default) is a common source of bugs when an API supports more than one representation format."
check_questions:
  - q: "Why does a plain `GET /orders` request from a browser to a different origin usually not trigger a preflight, while `fetch('/orders', {method: 'POST', headers: {'Content-Type': 'application/json'}, ...})` does?"
    a: "A GET with no custom headers and no body qualifies as a \"simple request\" under the CORS spec, which browsers have allowed cross-origin since before CORS existed (plain forms could already do this), so no preflight is needed. A POST with a JSON content type falls outside the simple-request definition — JSON isn't one of the three allowed simple content types — so the browser sends an OPTIONS preflight first to confirm the server actually accepts that combination before sending the real request with its body."
  - q: "A server responds to a cross-origin request with `Access-Control-Allow-Origin: *` and the request includes cookies via `credentials: 'include'`. What happens?"
    a: "The browser blocks it. The CORS spec explicitly disallows a wildcard `Access-Control-Allow-Origin` on a credentialed request — the server must echo back the specific requesting origin and also send `Access-Control-Allow-Credentials: true`, precisely because a wildcard plus credentials would let any website on the internet make authenticated requests on a logged-in user's behalf."
  - q: "If a request fails CORS, did the server actually process it?"
    a: "Usually yes, for a simple request that didn't need a preflight — the server received it, ran its logic, and sent a response; the browser is what then withholds that response from the calling JavaScript because the response headers didn't authorize the origin. This is why a request can show as \"succeeded\" in server-side logs or in a tool like curl or Postman (which don't enforce CORS) while failing in the browser."
  - q: "How does content negotiation offer an alternative to URI versioning for evolving an API's response shape?"
    a: "Instead of exposing `/v1/orders` and `/v2/orders` as different paths, a single `/orders` endpoint can serve different representations based on the `Accept` header's media type — `Accept: application/vnd.myapi.v1+json` versus `application/vnd.myapi.v2+json` — keeping one canonical resource URI while still letting a client choose which version of the representation it wants, which is exactly the header/media-type versioning strategy `api-versioning-strategies` covers as the alternative to putting the version in the path."
deeper:
  - label: "Learn CORS In 6 Minutes"
    url: "https://www.youtube.com/watch?v=PNtFSVU-YTI"
related_formulas: []
---

## CORS is the browser's rule, not the server's or the network's

A request from `app.example.com` to `api.example.com` is cross-origin — different subdomain, different origin, as far as the browser's same-origin policy is concerned, even though both are "your" infrastructure. Before CORS existed, the browser's default was to block cross-origin reads entirely, as a security boundary: without it, a malicious page could silently make authenticated requests to your bank's API using your logged-in session cookies and read the response. **CORS (Cross-Origin Resource Sharing)** is the mechanism that lets a server explicitly opt back into allowing specific cross-origin callers, via a set of `Access-Control-*` response headers.

The detail that trips up almost everyone the first time they hit a CORS error: it is enforced by the browser, on the response, not by the server refusing to process the request. For a plain `GET` with no custom headers, the request is sent, the server runs its normal logic, generates a response — and only then does the browser check whether that response's headers authorize the calling page's origin to actually read it. If they don't, the browser throws the error and the JavaScript never sees the response body, but the server-side logic already ran. This is exactly why a request that "fails with CORS" in the browser console will often show up as a completed 200 in server logs, and why testing the same request with `curl` or Postman — neither of which enforces CORS, because it's a browser behavior, not an HTTP one — succeeds with no error at all.

## When a preflight fires

Not every cross-origin request gets this treatment identically. The CORS spec defines a category of **simple requests** — a `GET`, `HEAD`, or `POST` with no custom headers and a `Content-Type` of `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` — that browsers allow to go straight through, because an HTML `<form>` could already make equivalent requests cross-origin before CORS existed, so there's no new capability to gate. Anything outside that set — a `PUT`, `PATCH`, or `DELETE`; a `POST` with `Content-Type: application/json` (the default for essentially every modern API); or any custom header like `Authorization` or `X-API-Key` — is **non-simple**, and the browser sends an automatic `OPTIONS` request first, called a **preflight**, before it will send the real one.

The preflight asks the server, in effect, "if I were to send this exact method with these exact headers, would you allow it?" via the `Access-Control-Request-Method` and `Access-Control-Request-Headers` request headers. The server answers with `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and `Access-Control-Allow-Origin` in the preflight response. If those permit what was asked, the browser proceeds to send the actual request; if not, it stops there and the real request never goes out at all — a `PUT /orders/482` that fails preflight never reaches the server's `PUT` handler. `Access-Control-Max-Age` lets the server tell the browser how long it can cache a preflight's result, so a client making repeated calls to the same endpoint with the same method and headers doesn't re-preflight every single time.

## The headers that actually grant access

`Access-Control-Allow-Origin` is the one that matters most and the one that produces the most common error: it must contain either the exact origin that made the request (scheme, host, and port all have to match — `https://app.example.com` and `http://app.example.com` are different origins) or the literal `*` wildcard for a fully public endpoint with no credentials involved. A mismatched or missing value here — serving the wrong environment's allowed origin in production, for instance — is responsible for the overwhelming majority of real-world CORS incidents, independent of whatever else is configured correctly.

**Credentialed requests** — ones that send cookies or HTTP auth via `credentials: 'include'` (fetch) or `withCredentials: true` (XHR) — carry an extra constraint deliberately: the server must echo the exact requesting origin, never a wildcard, and must additionally send `Access-Control-Allow-Credentials: true`. The spec forbids `*` plus credentials together on purpose. If it were allowed, any site on the internet could issue a credentialed request to an API serving `Access-Control-Allow-Origin: *`, and the browser would happily attach the logged-in user's session cookie to it — the wildcard-plus-credentials ban exists specifically to stop that. A CORS configuration that reflects *any* requesting origin back (common in permissive development setups) combined with allowing credentials is functionally the same hole, which is why it's worth treating "allow credentials" as something to turn on only for an explicit, known allowlist of origins, never a dynamic reflect-anything rule left over from local development.

## Content negotiation: agreeing on the shape of a response

Content negotiation is a separate HTTP mechanism that happens to share the same `Accept`/`Content-Type` header pair that shows up in CORS preflight checks, and it's worth being precise about the difference between the two headers themselves. `Content-Type`, sent on a request with a body, describes what the client is sending — `application/json`, `application/x-www-form-urlencoded`. `Accept`, sent on any request, describes what representations the client is willing to receive back — `Accept: application/json` says "respond with JSON if you can"; a server supporting multiple formats picks one that satisfies the client's `Accept` header and returns the matching `Content-Type` on the response, or a `406 Not Acceptable` if it can't satisfy any of the client's listed preferences.

The same mechanism extends past simple format selection into versioning. Instead of (or alongside) URI-based versioning, a server can key off a custom media type embedded in `Accept`: `Accept: application/vnd.myapi.v2+json` tells the server which version of the `/orders` representation the client wants, while the URI itself (`/orders`, not `/v2/orders`) stays stable. `api-versioning-strategies`, earlier in this group, covers this as header/media-type versioning and weighs it against URI versioning in full — the point worth connecting here is that it's the same `Accept`-header machinery already in play for ordinary format negotiation, just carrying a version number as part of the media type instead of (or in addition to) a format like `json` or `xml`. An API that already does content negotiation for format gets media-type versioning close to free; one that's never implemented content negotiation at all has more plumbing to add before that path is available.
