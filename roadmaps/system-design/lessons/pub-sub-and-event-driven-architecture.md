---
title: "Pub/Sub and Event-Driven Architecture"
description: How publish-subscribe differs from point-to-point queues — fan-out to many consumers, event buses, and choreography vs orchestration.
roadmap: system-design
node: pub-sub-and-event-driven-architecture
group: async-systems-and-messaging
video:
  provider: youtube
  id: "Zqvre6dWRw4"
  start: 0
  end: null
creator:
  name: CodeOpinion
  channel_url: https://www.youtube.com/@CodeOpinion
duration_seconds: 681
level: intermediate
prerequisites:
  - message-queues-and-async-communication
takeaways:
  - A point-to-point queue hands each message to exactly one consumer; publish-subscribe hands each message to every subscriber listening for it, which is a fundamentally different delivery guarantee, not just a configuration option on the same thing.
  - Publish-subscribe is the natural model for broadcasting that something happened — an event — to however many independent systems care about it, without the publisher knowing who they are or how many there are.
  - "Fan-out is what makes this powerful: one published event can be consumed by a notifications service, an analytics pipeline, and a fraud-detection system at the same time, each independently, each at its own pace."
  - An event bus or broker (Kafka, SNS, or similar) is the piece of infrastructure that actually tracks topics and subscribers and does the fan-out delivery, so publishers and subscribers never talk to each other directly.
  - Choreography coordinates a multi-step business process by having each service react to events and emit its own events in turn, with no central controller; orchestration uses a central coordinator that explicitly tells each step what to do next.
  - Choreography keeps services decoupled but makes the overall workflow harder to see in one place; orchestration makes the workflow explicit and easy to trace but reintroduces a central point of coordination — the deeper mechanics of managing failure across either style is its own topic (sagas), not this one.
check_questions:
  - q: A payment service publishes an "order placed" event. The shipping service, the email service, and the analytics pipeline all need to react to it. Why is publish-subscribe the right delivery model here instead of a point-to-point queue?
    a: A point-to-point queue delivers each message to exactly one consumer, so if all three services pulled from the same queue, only one of them would ever get any given event; publish-subscribe delivers a copy of the event to every subscriber, so all three can independently receive and react to the same "order placed" event without competing for it.
  - q: What does an event bus or broker actually do that a publisher and subscriber couldn't just do directly between themselves?
    a: It decouples them completely — the publisher sends the event to the broker's topic without knowing how many subscribers exist or who they are, and the broker handles delivering a copy to each current subscriber, so publishers and subscribers can be added, removed, or restarted independently without either side needing to track the other.
  - q: In choreography, how does a multi-step process like "place order, reserve inventory, charge payment, ship" actually get coordinated if there's no central controller?
    a: Each service listens for the event that signals it's its turn, does its piece of the work, and publishes its own event when it's done — the inventory service reacts to "order placed," the payment service reacts to "inventory reserved," and so on — so the sequence emerges from each service reacting to the previous step's event rather than from anything directing the whole flow.
deeper:
  - label: "Message Ordering in Pub/Sub or Queue"
    url: "https://www.youtube.com/watch?v=ILEb5LsSf5w"
related_formulas: []
---

## One message, many recipients

The previous lesson on message queues introduced point-to-point delivery: a producer writes a message, and exactly one consumer — whichever is free — picks it up and processes it. That's the right model for distributing *work*, where you want each job done once. But a lot of what happens in a real system isn't a job to be claimed — it's a fact to be announced. When an order is placed, that's not one unit of work for one worker; it's something that shipping, billing, analytics, and fraud detection all independently need to know about. **Publish-subscribe** (pub/sub) is the delivery model built for exactly that: every subscriber listening for a given message gets its own copy, not just one lucky winner.

```
Point-to-point:                    Publish-subscribe:

Producer → [queue] → Worker A      Publisher → [topic] → Subscriber A
                   (one consumer                       → Subscriber B
                    gets the job)                       → Subscriber C
                                   (every subscriber gets a copy)
```

This is a different *guarantee*, not a different setting on the same underlying thing. A point-to-point queue is designed so two consumers never both get the same message. A pub/sub topic is designed so every subscriber does.

## Fan-out: the point of the whole pattern

That one-to-many delivery is called **fan-out**, and it's the actual payoff of pub/sub. A single "order placed" event, published once, can simultaneously trigger a notification email, a row in a reporting warehouse, a fraud-risk score, and an update to a recommendation model — each handled by a completely separate system, each consuming the event on its own schedule, with none of them blocking the others and none of them even aware the others exist.

Crucially, the publisher doesn't enumerate any of this. It doesn't know the email service exists, doesn't know how many subscribers there are today, and doesn't need to change when a sixth system starts caring about order events tomorrow. That new system just subscribes to the existing topic. This is what makes pub/sub such a good fit for growing systems: adding a new consumer of an event is a purely additive change, with zero coordination required with the publisher or with any of the other existing subscribers.

## The broker does the actual fan-out

None of this happens by publishers calling subscribers directly — that would mean every publisher needs to know every subscriber's address, which is exactly the tight coupling pub/sub exists to avoid. Instead, an **event bus** or **broker** sits in the middle: Kafka, AWS SNS, Google Pub/Sub, and similar systems all play this role. The publisher sends its event to a named **topic** on the broker; the broker is the thing that knows who's currently subscribed to that topic and handles delivering a copy to each of them.

```
Publisher ──▶ Topic: "order.placed" ──▶ fan-out ──▶ Subscriber A
                                                  ──▶ Subscriber B
                                                  ──▶ Subscriber C
```

This is also what makes the "add a consumer without touching the publisher" property real in practice: a new subscriber just registers its interest in the topic with the broker. The publisher's code, and its deployment, never has to change.

## Choreography vs. orchestration

Pub/sub creates a natural way to coordinate a multi-step business process without a central controller: each step reacts to the event the previous step produced, and emits its own event when it's done. This style is called **choreography** — there's no conductor, just services that know how to respond to the events they care about, the way dancers in a choreographed routine each know their own part without anyone calling out instructions mid-performance.

```
"order placed" → inventory service reacts → emits "inventory reserved"
"inventory reserved" → payment service reacts → emits "payment charged"
"payment charged" → shipping service reacts → emits "order shipped"
```

The alternative is **orchestration**: a central coordinator explicitly calls each step in sequence and tracks where the overall process is. Orchestration makes the workflow visible in one place and easy to reason about — you can look at the orchestrator and see exactly what step a given order is on. Choreography keeps every service more decoupled — none of them need to know about the overall process, just the one event they react to — but that same property makes the *overall* flow harder to see, since it only exists as the sum of independent reactions scattered across services. Which style to use, and how to handle a step failing partway through either one, is the subject of its own topic — sagas — that goes well beyond what matters here: that pub/sub is what makes choreography possible in the first place, by giving every step a way to announce what just happened without knowing or caring who's listening.
