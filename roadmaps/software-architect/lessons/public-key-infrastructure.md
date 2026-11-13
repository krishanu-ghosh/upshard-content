---
title: "Public Key Infrastructure"
description: "Certificate authorities, trust chains, and revocation — the infrastructure of issuance and trust behind every certificate a system presents."
roadmap: software-architect
node: public-key-infrastructure
group: security-architecture
video:
  provider: youtube
  id: "CD1mQXyPOuY"
  start: 0
  end: null
creator:
  name: CyberPlatter
  channel_url: https://www.youtube.com/@cyberplatter
duration_seconds: 354
level: intermediate
prerequisites: []
takeaways:
  - "PKI is the organizational machinery around a key pair — the certificate authorities, trust chains, and revocation systems that let a stranger's public key be trusted, not the cryptography itself."
  - "A certificate authority's signature on a certificate is what turns a bare claim of \"here is a public key\" into a vouched one — a CA that browsers already trust is asserting that this specific key belongs to this specific organization."
  - "Trust chains up: a leaf certificate is signed by an intermediate CA, which is signed by a root CA whose public key ships pre-installed in the operating system or browser — verifying a certificate means walking that chain back to something already trusted."
  - "A certificate is only as trustworthy as its revocation story — CRLs and OCSP exist because a compromised private key has to be invalidated before its certificate's expiry date, not just left to expire naturally."
  - "An organization can run its own private CA for internal use — issuing certificates for service-to-service mTLS, VPN clients, or code signing — without ever touching a public CA, as long as every internal party is configured to trust that private root."
check_questions:
  - q: What does a certificate authority's signature on a certificate actually assert?
    a: It asserts that the CA verified the identity of the entity requesting the certificate and is vouching that the public key inside the certificate really belongs to that entity — not that the key itself is cryptographically special in any way.
  - q: Why does verifying a certificate mean walking a chain rather than checking one signature?
    a: Browsers and operating systems only ship pre-installed trust for a small set of root CAs; a server's leaf certificate is usually signed by an intermediate CA, so the verifier has to follow signatures up from the leaf through each intermediate until it reaches a root it already trusts.
  - q: Why isn't a certificate's expiry date enough to handle a compromised private key?
    a: An attacker who steals a private key can impersonate its owner for however long is left until expiry, which could be months; CRLs and OCSP let a CA say "this certificate is no longer valid" immediately, rather than forcing everyone to wait out the original validity period.
deeper: []
related_formulas: []
---

## The problem PKI solves

Asymmetric cryptography gives you a key pair: a private key you never share, and a public key anyone can use to verify your signature or encrypt something only you can read. But a bare public key is just a string of bytes — nothing about it says who it belongs to. If an attacker hands you *their* public key and claims to be your bank, the math works perfectly; you've just encrypted your data to the wrong person. **Public key infrastructure** is the set of roles, certificates, and procedures that bind a public key to a verified identity, so "here is a public key" becomes "here is a public key, and someone you already trust vouches that it belongs to this specific organization."

## Certificates and the chain of trust

A **digital certificate** (almost always in X.509 format) packages a public key together with identity information — a domain name, an organization name — and a signature from a **certificate authority (CA)** that attests the pairing is real. The CA doesn't just take your word for it: before issuing a certificate, it runs a validation step appropriate to the certificate's level — domain validation (DV) just confirms you control the domain, while organization validation (OV) and extended validation (EV) check real-world business registration too.

Trust doesn't stop at one CA's signature, because no browser or OS could keep an up-to-date trust list for every CA on earth. Instead, trust is **chained**: a server's certificate (the "leaf") is typically signed by an **intermediate CA**, and the intermediate's own certificate is signed by a **root CA**. Root CA public keys are the ones actually baked into your operating system or browser's trust store — a small, carefully curated set, because compromising a root CA would let an attacker mint trusted certificates for anything. Verifying a certificate means walking this chain: check the leaf's signature against the intermediate's public key, check the intermediate's signature against the root's public key, and confirm the root is one you already trust. If every link holds, the chain is valid; if any signature doesn't match or the chain dead-ends at an unrecognized root, verification fails — which is exactly the "your connection is not private" warning a browser shows for a self-signed or misconfigured certificate.

This is the trust infrastructure that sits *underneath* the TLS handshake — `tls-and-https` (in `computer-science`) covers how a client and server actually use a certificate during the handshake itself; this lesson is about where that certificate came from and why it can be trusted at all.

## Revocation: the part expiry dates don't cover

Certificates carry an expiry date, usually a year or less for public TLS certificates, which limits how long a compromised key stays dangerous by default. But a year is a long time to leave a stolen private key valid. If a CA discovers a certificate should no longer be trusted — the private key leaked, the organization's domain was reassigned, the certificate was mis-issued — it needs a way to say so immediately, not wait for the expiry date. Two mechanisms exist for this:

- **Certificate Revocation Lists (CRLs)** are periodically published lists of revoked certificate serial numbers, signed by the CA. A verifier downloads the list and checks whether the certificate it's validating appears on it.
- **Online Certificate Status Protocol (OCSP)** replaces the bulk download with a live, per-certificate query: the verifier asks the CA (or a responder acting on its behalf), "is this specific serial number still valid?" and gets a signed yes/no back in real time.

OCSP's live query has its own cost — it adds a network round-trip to every connection and gives the CA a record of who's checking which certificate. **OCSP stapling** fixes both: the server itself periodically fetches a signed, time-stamped OCSP response and "staples" it to the TLS handshake, so the client gets revocation proof without a separate query and without revealing its browsing to the CA.

## Running PKI inside your own organization

Nothing about PKI requires a public CA. An organization can stand up its own **private root CA** and issue certificates signed by it for purely internal use — service-to-service mutual TLS (mTLS) inside a cluster, VPN client authentication, or signing internal build artifacts and code. The one requirement is that every party who needs to trust those certificates has the private root's public key installed in its own trust store ahead of time, the same way public roots ship pre-installed in browsers. This is how a service mesh (see `system-design`'s `sidecar-ambassador-and-service-mesh`) typically authenticates every service-to-service connection: a private internal CA, often rotating short-lived certificates automatically, rather than hand-issued long-lived ones.

## What an architect actually decides here

The architectural decisions in PKI aren't about the cryptography — they're about **operational ownership**: whether to use a public CA (simpler, but no control over issuance policy or revocation speed) or run a private CA (full control, but you now own certificate lifecycle management for everything that depends on it); how short to make certificate lifetimes, trading operational overhead (more frequent rotation) against the blast radius of a compromised key; and whether revocation checking is even enforced strictly, since many clients historically "soft-fail" an unreachable OCSP responder rather than block the connection, which quietly weakens the whole revocation story. Getting these defaults right, once, at the architecture level is cheaper than discovering them during an incident.
