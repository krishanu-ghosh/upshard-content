---
title: "Client and Browser Caching"
description: How Cache-Control, ETags, and service workers let the client itself skip the network entirely — the layer closest to the user.
roadmap: system-design
node: client-and-browser-caching
group: caching-at-scale
video:
  provider: youtube
  id: "TgZnpp5wJWU"
  start: 0
  end: null
creator:
  name: Hussein Nasser
  channel_url: https://www.youtube.com/@hnasr
duration_seconds: 1007
level: intermediate
prerequisites:
  - caching-and-cdns
takeaways:
  - A browser cache is the layer closest to the user in the stack this roadmap's caching-and-CDNs lesson already laid out — a hit here costs zero network round trips, which is strictly cheaper than even the nearest CDN edge node.
  - The Cache-Control header controls freshness directly — a directive like max-age tells the browser how long it can reuse a response without asking the server anything at all, no network request involved.
  - Once a cached response's freshness window expires, the browser doesn't necessarily have to redownload it — a conditional request (If-None-Match with an ETag, or If-Modified-Since with a Last-Modified date) lets the server reply with a tiny 304 Not Modified instead of resending the full body.
  - An ETag is an opaque fingerprint of a specific version of a resource; it changes whenever the underlying content changes, which makes it a more precise validator than a last-modified timestamp for content that can change without its timestamp being reliable.
  - A service worker runs as a programmable proxy inside the browser, giving an application explicit control over which requests are served from a cache, which go to the network, and what happens when the network isn't available at all — a layer of control plain HTTP caching headers can't offer.
check_questions:
  - q: Why is max-age in a Cache-Control header more efficient than a conditional request for a fresh resource?
    a: Within the max-age window, the browser treats its cached copy as valid without making any request at all, whereas a conditional request still requires a full round trip to the server — max-age avoids the network entirely, while a conditional request only avoids re-sending the response body.
  - q: What does an ETag let a server avoid sending that it would otherwise have to send?
    a: When a client's cached copy is still current, the server can respond with a 304 Not Modified and an empty body instead of resending the entire resource, because the client's If-None-Match header already told the server exactly which version the client has cached.
  - q: What can a service worker do that HTTP cache headers alone cannot?
    a: A service worker can run custom logic for every request an application makes — deciding programmatically whether to serve from a cache, go to the network, race both, or fall back to a cached response when there's no network connection at all — which goes well beyond what a declarative Cache-Control or ETag policy can express on its own.
related_formulas: []
---

## The layer that costs nothing when it hits

The caching-and-CDNs lesson already laid out the stack a request passes through — browser, CDN, reverse proxy, application, database — and made the point that the closer a cache sits to the user, the cheaper a hit is. Client and browser caching is that stack's first and cheapest layer: when it works, the browser serves a response straight from local storage, without a network request ever leaving the device. This lesson is about the three mechanisms that make that possible and controllable: HTTP cache headers, conditional requests, and service workers.

## Cache-Control: declaring freshness up front

The **Cache-Control** response header is how a server tells the browser (and any cache sitting between them) how to treat a given response. The most important directive is `max-age`, which states, in seconds, how long the response can be reused without checking back with the server at all:

```
Cache-Control: max-age=86400
```

For the next 24 hours, the browser can serve this exact response straight out of its cache for any matching request, with zero network traffic involved — not even a request to ask "is this still good?" That's the cheapest possible cache hit there is. Other directives shape this behavior: `no-store` forbids caching the response at all (appropriate for anything sensitive or strictly per-request); `no-cache` is a confusingly-named directive that actually *does* allow caching but forces a revalidation check with the server before reusing it every time; `private` marks a response as cacheable only in the end user's own browser, not in a shared cache like a CDN or proxy sitting between many users — important for anything personalized.

Setting these correctly is a judgment call based on how often content actually changes and how bad it is to serve something stale: an immutable, content-hashed JavaScript bundle (`app.a3f29c.js`) can have an extremely long `max-age` because its filename itself changes whenever its content does, while an API response for a user's current balance probably shouldn't be cached by a shared cache at all.

## Conditional requests: cheap revalidation once freshness expires

Once a cached response's `max-age` window has passed, the browser doesn't have to assume the content changed — it just has to check. A **conditional request** lets it ask "has this changed since I last saw it?" without forcing a full re-download if the answer is no.

The two validators are a **Last-Modified** timestamp and an **ETag**. A Last-Modified date is simple but coarse — it can't distinguish two different versions saved within the same second, and it depends on the server correctly tracking modification times. An **ETag** is an opaque fingerprint (often a hash) identifying one specific version of a resource's content, which makes it precise regardless of timestamps:

```
GET /api/users/42
If-None-Match: "33a64df551"
```

If the resource's current ETag still matches, the server replies:

```
HTTP/1.1 304 Not Modified
```

with no response body at all — the browser is told its cached copy is still correct and reuses it. If the content has changed, the server sends a normal `200 OK` with the full new body and a new ETag. Either way, this round trip is far cheaper than re-sending a full response every time a cache entry's freshness window lapses, which is exactly why conditional requests matter even for resources you don't want to cache indefinitely with a long `max-age`.

## Service workers: programmable caching, not just declarative headers

Cache-Control and ETags are declarative — the server states a policy, and the browser follows it according to fixed rules. A **service worker** is something different: a script that the browser runs in the background, separate from any page, that can intercept every network request a web application makes and decide programmatically what happens to it.

```javascript
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      return cached || fetch(event.request).then(response => {
        const copy = response.clone();
        caches.open('v1').then(cache => cache.put(event.request, copy));
        return response;
      });
    })
  );
});
```

This snippet implements a cache-aside-style strategy entirely in the client: check the service worker's own cache storage first, and only hit the network on a miss, saving a copy of the response for next time. Because this is ordinary code rather than a fixed header-driven policy, a service worker can express strategies HTTP caching headers can't — serve from cache first and update in the background (stale-while-revalidate, implemented by hand), race the cache and the network and use whichever responds first, or fall back to a cached shell page when there's no network connection at all, which is what makes offline-capable web applications possible.

## Where this sits in the bigger picture

Client and browser caching is the cheapest cache layer precisely because it eliminates the network round trip entirely rather than just shortening it — a CDN edge hit still costs one trip to a nearby server, while a browser cache hit costs nothing. But it's also the layer with the least centralized control: a server can set policy through headers and expect browsers to follow it, and a service worker gives an application explicit control over its own client's behavior, but neither gives an operator the kind of direct visibility or override a CDN or reverse-proxy cache has. That's the tradeoff worth remembering when deciding how much to lean on this layer versus the ones behind it: it's the fastest possible cache hit, and also the one furthest from your own infrastructure's control.
