---
title: "The OSI Model and TCP/IP Model"
description: How networking is split into layers, what each layer is responsible for, and why TCP/IP — not OSI — is what the internet actually runs on.
roadmap: computer-science
node: osi-and-tcp-ip-models
group: networking-fundamentals
video:
  provider: youtube
  id: "CRdL1PcherM"
  start: 0
  end: null
creator:
  name: NetworkChuck
  channel_url: https://www.youtube.com/@NetworkChuck
duration_seconds: 720
level: beginner
prerequisites: []
takeaways:
  - Layering splits "get a message from one program to another, over a wire or through the air" into independent jobs, so a change at one layer — a new physical medium, a new routing protocol — doesn't force a rewrite of the layers above it.
  - The OSI model has seven layers (Physical, Data Link, Network, Transport, Session, Presentation, Application); the practical shorthand most engineers reach for is "layer 2" (switches, MAC addresses), "layer 3" (routers, IP addresses), "layer 4" (TCP/UDP, ports), and "layer 7" (the application protocol itself, like HTTP).
  - TCP/IP, the model actually implemented on the internet, collapses this into four layers (Link, Internet, Transport, Application) and was designed from working code rather than from a committee's diagram.
  - OSI survives as a teaching and troubleshooting reference — "is this a layer 1 problem (a bad cable) or a layer 7 problem (a broken API)?" — even though no real network stack implements all seven layers as separate, distinct protocols.
  - Every layer adds its own header to the data handed down from the layer above (encapsulation) and strips it back off on the way up at the receiving end, so a layer 4 segment is wrapped inside a layer 3 packet, which is wrapped inside a layer 2 frame.
check_questions:
  - q: Why does networking get split into layers instead of being one monolithic piece of software?
    a: Splitting into layers means each layer only has to solve one job — addressing, routing, reliable delivery, or application semantics — so a change at one layer, like swapping Wi-Fi for Ethernet, doesn't require touching the layers above it that don't care how the bits physically moved.
  - q: What's the practical difference between a "layer 3 problem" and a "layer 7 problem" when debugging a network issue?
    a: A layer 3 problem is about routing and addressing — packets aren't reaching the right IP address at all — while a layer 7 problem means packets are arriving fine but the application protocol riding on top (like a malformed HTTP request) is broken.
  - q: Why does TCP/IP have four layers instead of OSI's seven?
    a: TCP/IP grew out of working protocols built to solve real problems, and several of OSI's distinctions (Session, Presentation) turned out not to need their own protocol layer in practice, so TCP/IP merged them into its Application layer.
  - q: What does "encapsulation" mean in this context?
    a: Each layer wraps the data it receives from the layer above inside its own header before passing it down, so an HTTP request ends up wrapped in a TCP segment, which is wrapped in an IP packet, which is wrapped in an Ethernet frame by the time it leaves the machine.
deeper:
  - label: "Computer Networking in 100 Seconds"
    url: "https://www.youtube.com/watch?v=keeqnciDVOo"
related_formulas: []
---

## Why bother with layers at all

Getting a message from a program on one computer to a program on another computer is a deceptively large problem. The message has to be broken into a form that can actually travel over a wire or through radio waves; it has to be addressed so the right machine on a network of millions receives it; it has to be delivered reliably even though individual packets get lost, duplicated, or arrive out of order; and once it arrives, some application has to know what to do with the bytes. Trying to solve all of that in one undifferentiated blob of code would mean that fixing how data gets physically transmitted (say, moving from Ethernet cables to Wi-Fi) risks breaking how applications interpret messages, even though the two concerns have nothing to do with each other.

The fix networking settled on is layering: split the problem into a stack of independent jobs, each with a narrow responsibility and a well-defined interface to the layers above and below it. A layer only needs to know how to talk to its immediate neighbors — it doesn't need to understand what's happening two layers up or down. That's what makes it possible for Wi-Fi, Ethernet, and cellular data to all carry the exact same web traffic: they're interchangeable implementations of the bottom layer, invisible to everything above.

## The OSI model's seven layers

The OSI (Open Systems Interconnection) model is the formal, academic version of this idea, and it names seven layers:

1. **Physical** — the actual transmission of raw bits as electrical signals, light pulses, or radio waves.
2. **Data Link** — getting frames reliably across a single physical link, using addressing (MAC addresses) that's local to that link.
3. **Network** — routing data across multiple interconnected networks using addresses that are globally meaningful (IP addresses).
4. **Transport** — end-to-end delivery between two programs, including whether that delivery is reliable and ordered (TCP) or not (UDP).
5. **Session** — establishing, managing, and tearing down a conversation between two applications.
6. **Presentation** — translating between the data format an application uses and the format sent over the network (encoding, compression, encryption).
7. **Application** — the protocol the actual program speaks, like HTTP, SMTP, or DNS.

In practice, almost nobody writes software that implements Session and Presentation as their own distinct protocol layers — those concerns got absorbed into the application layer or into libraries. But the seven-layer breakdown is still useful shorthand: when something's broken, "is this a layer 2 issue (a switch dropping frames), a layer 3 issue (routing), or a layer 7 issue (the application's own logic)?" is a genuinely useful diagnostic question, because it tells you which part of the stack to go investigate.

## The TCP/IP model: four layers, built from what actually shipped

TCP/IP — the suite of protocols the real internet runs on — predates OSI's formal seven-layer model and was never designed to match it. It groups the same concerns into four layers: **Link** (physical transmission and local addressing, merging OSI's Physical and Data Link), **Internet** (routing between networks, i.e. IP — matching OSI's Network layer), **Transport** (TCP and UDP, matching OSI's Transport layer), and **Application** (everything above that — HTTP, DNS, SMTP, and all the session/presentation concerns those protocols handle themselves).

The reason TCP/IP won isn't that four layers is inherently better than seven — it's that TCP/IP was built by implementing working protocols and refining them, while OSI was specified top-down by a standards committee before much of it was ever built. By the time OSI's seven-layer protocols were ready, the internet was already running on TCP/IP, and a working standard with a massive installed base beats a more theoretically complete one that arrives later.

## Encapsulation: how a layer actually talks to its neighbor

The mechanism that makes layering work is encapsulation. When an application sends data, each layer below it wraps that data in its own header before handing it further down: the application's bytes become the payload of a TCP segment (adding source/destination ports and sequencing info), the TCP segment becomes the payload of an IP packet (adding source/destination IP addresses), and the IP packet becomes the payload of an Ethernet frame (adding source/destination MAC addresses) before it's turned into electrical signals and sent. On the receiving end, each layer strips off its own header and hands the remainder up to the layer above, which is how a web server ends up with exactly the HTTP request the browser sent, with none of the addressing machinery visible to the application code at all.

That separation is also why you can run the exact same web application over Ethernet, Wi-Fi, or a cellular connection without changing a line of application code: everything below the Transport layer is free to vary, because each layer only has to honor its contract with its immediate neighbor.
