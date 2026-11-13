---
title: "DNS and HTTP: What Happens When You Hit Enter"
description: How a domain name becomes an IP address, and the request/response protocol that carries the page back once it has one.
roadmap: computer-science
node: dns-and-http
group: networking-fundamentals
video:
  provider: youtube
  id: "QHSsL0Seqhs"
  start: 0
  end: null
creator:
  name: Java Techie
  channel_url: https://www.youtube.com/@Javatechie
duration_seconds: 900
level: beginner
prerequisites:
  - osi-and-tcp-ip-models
takeaways:
  - A domain name is a human-friendly alias; before a browser can send anything, it has to resolve that alias to an IP address through DNS, a distributed, hierarchical lookup system.
  - DNS resolution walks down a hierarchy — a recursive resolver asks a root server where to find the top-level domain's servers, asks that TLD server where the domain's authoritative servers are, then asks an authoritative server for the actual IP address.
  - Once the browser has an IP address, it opens a TCP connection to that address and sends an HTTP request — a structured text message naming a method (GET, POST, …), a path, headers, and optionally a body.
  - The server's HTTP response carries a status code that summarizes the outcome in one number (200 for success, 404 for not found, 500 for a server error) along with its own headers and body.
  - DNS answers are cached at multiple levels (the browser, the OS, the ISP's resolver) with an expiry time (TTL) set by the domain's owner, which is why DNS changes can take time to "propagate" — caches elsewhere haven't expired yet.
check_questions:
  - q: Why can't a browser just send an HTTP request directly to "example.com"?
    a: Network routing works in terms of IP addresses, not names, so the browser has to resolve the domain name to an IP address through DNS before it can open a connection and send anything.
  - q: What role does a root server play in DNS resolution, given that it doesn't actually know the IP address being looked up?
    a: A root server doesn't store the final answer — it tells the resolver which servers are authoritative for the top-level domain (like .com), pointing the resolution process one level deeper into the hierarchy rather than answering it directly.
  - q: What does an HTTP status code like 404 or 500 actually tell the client?
    a: It's a compact summary of the outcome — 404 means the server understood the request but found nothing at that path, while 500 means the server itself hit an error trying to produce a response, which is a useful distinction for deciding what to do next.
  - q: Why might a DNS change you just made not be visible to everyone immediately?
    a: DNS answers are cached with a time-to-live set by the domain owner, and resolvers elsewhere (ISPs, other browsers) won't re-fetch the new answer until their cached copy's TTL expires, so the change "propagates" gradually rather than instantly.
deeper:
  - label: "DNS Explained in 100 Seconds"
    url: "https://www.youtube.com/watch?v=UVR9lhUGAyU"
related_formulas: []
---

## The question DNS answers

Every machine reachable over the internet has an IP address, but nobody memorizes `142.250.80.14` when they mean to visit a search engine — they type a name. DNS (the Domain Name System) exists to close that gap: it's a distributed database that maps human-readable domain names to the IP addresses machines actually need to route traffic. "Distributed" is the key word — no single server holds every mapping in the world. Instead, DNS is organized as a hierarchy, and resolving a name means walking down that hierarchy one step at a time.

## Walking the hierarchy

Say your browser needs the IP address for `www.example.com`. It hands the job to a **recursive resolver** — usually run by your ISP or a public service — whose job is to do all the asking on your behalf and hand back a final answer. The resolver doesn't know the answer either, so it starts at the top:

1. It asks a **root server** (there are only a handful of these, run by different organizations worldwide) "who handles `.com`?" The root server doesn't know where `example.com` lives, but it does know which servers are authoritative for the `.com` top-level domain, and it returns that pointer.
2. The resolver then asks one of those **TLD servers** "who handles `example.com`?" The TLD server likewise doesn't have the final IP address, but it knows which **authoritative name servers** are responsible for the `example.com` zone, and returns that.
3. Finally, the resolver asks that authoritative server directly: "what's the IP address for `www.example.com`?" This time it gets the real answer — an IP address — and hands it back to your browser.

Each of those answers gets cached along the way with a **TTL** (time to live) set by the domain's owner, so the next request for the same name doesn't have to repeat the whole walk. That caching is also why DNS changes don't take effect everywhere instantly: a resolver somewhere else in the world might be holding onto a cached answer until its TTL expires, even though the authoritative record has already changed — what people call "DNS propagation" is really just caches expiring on their own schedules.

## Now the browser has an address — what next?

With an IP address in hand, the browser opens a TCP connection to the server (and, for HTTPS, performs a TLS handshake on top of that connection — the subject of the next lesson). Then it sends an **HTTP request**: a structured text message with a **method** that states the intent (`GET` to retrieve something, `POST` to submit data, and others like `PUT`, `DELETE`, `PATCH`), a **path** identifying the resource (`/search?q=…`), and a set of **headers** carrying metadata (what content types the client accepts, cookies, authentication tokens). It can also carry a **body** with data to send — GET requests normally omit this, POST requests typically include it.

## The response, and what its status code is really saying

The server processes the request and sends back an **HTTP response**: a **status code**, its own headers, and usually a body (the HTML, JSON, or other content being returned). The status code is a compact, three-digit summary of what happened, and the first digit tells you the category before you even look at the rest. **2xx** means success (`200 OK` is the everyday case). **3xx** means the client should look elsewhere (a redirect). **4xx** means the client's request was the problem (`404 Not Found` — there's nothing at that path; `401 Unauthorized` — you need to authenticate). **5xx** means the server itself failed while trying to handle an otherwise valid request (`500 Internal Server Error`).

That distinction between 4xx and 5xx matters in practice: a 404 tells a client "stop asking, that resource genuinely isn't there," while a 500 tells a client "something broke on our end, and retrying might actually work." Logging and alerting systems lean on exactly that difference — a spike in 5xx responses usually means your own service is unhealthy, while a spike in 4xx responses more often means clients are requesting things that don't exist, which is a different kind of problem entirely.

## Putting it together

Typing a URL and hitting enter chains both halves of this lesson: DNS resolves the name to an address through a hierarchy of servers that each know one level more of the answer, then HTTP carries a request to that address and a response back, with a status code that tells the browser — and you — exactly what happened. Every web request you'll ever debug starts by figuring out which of these two steps actually failed: did the name even resolve, or did it resolve fine and the HTTP exchange itself go wrong?
