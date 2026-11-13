---
title: "TLS and HTTPS"
description: How HTTPS gets its privacy and integrity guarantees from TLS — the handshake, certificates, and what encryption in transit does and doesn't protect.
roadmap: computer-science
node: tls-and-https
group: networking-fundamentals
video:
  provider: youtube
  id: "E_wX40fQwEA"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 720
level: intermediate
prerequisites:
  - dns-and-http
takeaways:
  - HTTPS is plain HTTP carried over a TLS-encrypted connection; the request and response format doesn't change, only the transport underneath it.
  - TLS solves three problems at once — confidentiality (nobody in between can read the traffic), integrity (nobody in between can silently modify it), and authentication (you're actually talking to the server you think you are).
  - The TLS handshake uses asymmetric cryptography to safely agree on a shared secret key over an unencrypted channel, then switches to much faster symmetric encryption for the actual data using that shared key.
  - A certificate is a signed statement binding a public key to a domain name; browsers trust it because it's signed by a certificate authority whose own key is already built into the browser or operating system.
  - TLS protects data in transit between client and server, but it says nothing about what happens to that data once it arrives — a server can still log it, leak it, or get breached after a perfectly secure TLS session delivered it.
check_questions:
  - q: What's the actual difference between an HTTP request and an HTTPS request, in terms of the data sent?
    a: There isn't one at the HTTP layer — HTTPS sends the exact same method, path, headers, and body as HTTP, but the whole exchange travels over a TLS-encrypted connection instead of a plaintext one, so the content is hidden from anyone observing the raw network traffic.
  - q: Why does TLS use slow asymmetric cryptography only during the handshake instead of for the whole connection?
    a: Asymmetric cryptography is computationally expensive, so TLS uses it only briefly to safely agree on a shared secret key over a connection that starts out unencrypted, then switches to fast symmetric encryption — which needs that shared key — for the actual back-and-forth of request and response data.
  - q: What is a certificate actually asserting, and why does a browser trust it?
    a: A certificate asserts that a specific public key belongs to a specific domain name, and the browser trusts that assertion because it's digitally signed by a certificate authority whose own public key is already built into the browser or operating system as a trust anchor.
  - q: If a site uses HTTPS, does that mean the data you submit to it is safe?
    a: Only in transit — TLS guarantees nobody intercepting the connection can read or tamper with the data on its way to the server, but it says nothing about how securely the server itself stores, processes, or protects that data afterward.
deeper:
  - label: "How SSL, TLS, & HTTPS Work | Keeping Your Connections Safe"
    url: "https://www.youtube.com/watch?v=a6KW_ImFgU0"
related_formulas: []
---

## What HTTPS actually adds

HTTPS is not a different protocol from HTTP in the sense of changing what gets sent — the methods, paths, headers, and bodies are identical. What changes is the transport underneath: instead of a plain TCP connection carrying HTTP text that anyone on the network path could read or alter, HTTPS runs HTTP over a **TLS** (Transport Layer Security) connection, which wraps that traffic in encryption before it ever leaves the machine. Strip away the acronym and TLS is solving three distinct problems that plain HTTP leaves wide open:

- **Confidentiality** — someone capturing the traffic on a shared Wi-Fi network, or at any router along the path, shouldn't be able to read the contents.
- **Integrity** — nobody in between should be able to quietly modify the request or response without either side noticing.
- **Authentication** — the client needs some way to know it's actually talking to the real `example.com` server, not an impostor intercepting the connection.

## Why the handshake uses two different kinds of cryptography

Solving confidentiality looks simple on paper: encrypt everything with a shared secret key, so only the two parties who know the key can read the traffic. The hard part is how client and server agree on that secret key in the first place, when the only channel they have to communicate is the same unencrypted network a third party might already be watching. Sending the key itself over that channel in the clear would defeat the entire point.

This is where **asymmetric cryptography** earns its cost. In an asymmetric scheme, each party has a public key (safe to share with anyone) and a private key (never shared). Data encrypted with the public key can only be decrypted with the matching private key. That property lets the client and server agree on a shared secret even while an eavesdropper watches every message go by: the client can use the server's public key to encrypt key material that only the server's private key can unlock, and neither side ever has to transmit the secret itself in a form an observer could use.

Asymmetric cryptography is computationally expensive, though — far too slow to encrypt every byte of every HTTP request and response for the life of a connection. So TLS uses it only for this brief handshake, to establish a shared **symmetric** key, and then switches to symmetric encryption (which is orders of magnitude faster) for the actual application data that follows. That's the shape of almost every modern TLS handshake: a short, expensive asymmetric negotiation to bootstrap a symmetric key, followed by a long, cheap symmetric session using it.

## Certificates: proving who you're actually talking to

Agreeing on a key with *somebody* doesn't help if that somebody is an attacker impersonating the real server — a classic man-in-the-middle. TLS solves authentication with **certificates**. A certificate is a signed document stating "this public key belongs to `example.com`," and the signature comes from a **certificate authority** (CA), an organization whose job is to verify that whoever requested the certificate actually controls that domain before signing it.

The reason a browser trusts that signature at all is that a short list of CA public keys is already baked into the browser or operating system as **trust anchors**. When a server presents its certificate during the handshake, the browser checks that it's signed by a CA it already trusts (possibly through a chain of intermediate certificates), confirms the domain name matches, and checks it hasn't expired or been revoked. Only after all of that does the browser proceed with the key exchange — the little padlock icon is a summary of that whole check having passed, not a guarantee about anything beyond it.

## What TLS doesn't cover

It's worth being precise about the boundary here, because "the site uses HTTPS" gets treated as a blanket reassurance it was never meant to be. TLS protects data **in transit** — between your browser and the server's edge. It says nothing about what the server does with that data once it arrives: whether it's logged in plaintext somewhere, stored in a database with weak access controls, or exposed by an unrelated vulnerability in the server's own code. A password submitted over a flawless TLS connection to a server that stores passwords in plaintext is still a password that will eventually leak. TLS closes off one specific class of attack — interception on the wire — and leaves every question about server-side security exactly as open as it was before.
