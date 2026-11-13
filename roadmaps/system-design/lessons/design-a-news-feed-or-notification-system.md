---
title: "Case Study: Design a News Feed / Notification System"
description: A worked design for a social feed and notification pipeline, centered on the fan-out-on-write vs. fan-out-on-read decision.
roadmap: system-design
node: design-a-news-feed-or-notification-system
group: case-studies
video:
  provider: youtube
  id: "zR6XlMPZSYs"
  start: 0
  end: null
creator:
  name: TechGyanWithAvinash
  channel_url: https://www.youtube.com/@AkGautam_1904
duration_seconds: 2617
level: advanced
prerequisites:
  - pub-sub-and-event-driven-architecture
  - caching-and-cdns
takeaways:
  - A news feed is fundamentally a fan-out problem — every post has to reach every follower's feed, and the question is simply when that fan-out work happens.
  - Fan-out-on-write (push a new post into every follower's precomputed feed immediately) makes reads cheap but makes posting expensive, in proportion to the poster's follower count.
  - Fan-out-on-read (compute a feed by pulling recent posts from everyone a user follows, at read time) makes posting cheap but makes every feed load an expensive multi-way merge.
  - A celebrity account with millions of followers breaks pure fan-out-on-write, because pushing one post to millions of feeds synchronously (or even in a reasonable time window) doesn't scale.
  - The standard fix is a hybrid — fan out on write for ordinary accounts, but fan out on read (merged in at feed-load time) for accounts above some follower-count threshold.
  - Feed ranking is a separate concern from feed assembly — even a perfectly fanned-out feed still needs a scoring step to decide what order to show candidate posts in, since recency alone is rarely the best signal.
check_questions:
  - q: Why does fan-out-on-write make writes (posting) expensive specifically for accounts with many followers?
    a: Fan-out-on-write means every new post triggers an update to every single follower's precomputed feed, so the amount of work a single post generates scales linearly with the poster's follower count, which is fine for a typical user but becomes an enormous burst of writes for an account with millions of followers.
  - q: What is the main downside of fan-out-on-read compared to fan-out-on-write?
    a: Fan-out-on-read defers all the work to the moment a user opens their feed, requiring the system to fetch and merge recent posts from every account that user follows on the spot, which makes feed loads slower and more expensive, especially for users who follow a large number of accounts, trading cheap writes for expensive reads.
  - q: Why do most large-scale feed systems use a hybrid of fan-out-on-write and fan-out-on-read rather than picking just one?
    a: Pure fan-out-on-write collapses under celebrity accounts with huge follower counts, and pure fan-out-on-read makes every feed load expensive for everyone, so the hybrid approach fans out normally for typical accounts (keeping most reads cheap) while merging in celebrity posts at read time (avoiding an impossibly large write burst for their rare, high-fan-out posts).
deeper:
  - label: "Designing INSTAGRAM: System Design of News Feed (Gaurav Sen)"
    url: "https://www.youtube.com/watch?v=QmX2NPkJTKg"
related_formulas: []
---

## Requirements

A news feed (Instagram/Twitter-style) and a notification system (push alerts for likes, comments, follows) are really the same underlying problem wearing different clothes: both are about getting an event from one producer to a large, dynamic set of interested consumers, efficiently. This case study treats the feed as the primary example and notes where notifications diverge.

Functional requirements:
- A user can post content; the post should appear in followers' feeds.
- A user can open their feed and see recent, relevant posts from accounts they follow, roughly in ranked order.
- A user receives notifications for relevant events (someone liked their post, followed them, commented) without having to poll for them.

Non-functional requirements:
- **Low feed-load latency**: opening the app should show a feed in a few hundred milliseconds, not seconds.
- **High write throughput and extreme fan-out variance**: most accounts have a handful to a few thousand followers; a tiny fraction have tens of millions, and the system has to handle both without the rare case breaking the common one.
- **Eventual, not immediate, consistency is acceptable**: a follower seeing a new post a few seconds late is a non-issue; this buys real design flexibility.
- **Freshness-ranking balance**: a feed that's purely chronological is simple but often less engaging than one that weighs recency against relevance.

## Capacity estimate

Suppose the platform has 100 million daily active users, an average of 200 followers per account (heavily skewed — most accounts have far fewer, a small number have millions), and 50 million new posts/day.

- Average fan-out writes/day: 50M posts × 200 followers ≈ 10 billion feed-insertion events/day from an averaged-out perspective — but this average badly hides the real shape of the problem, since a single celebrity post with 20 million followers generates as much fan-out work alone as 100,000 average posts combined.
- Feed reads/day: 100M users × ~10 feed loads/day = 1 billion feed reads/day ≈ ~11,500/second average, several times that at peak — each one needs to return in well under a second.
- Precomputed-feed storage: caching the most recent ~500 post IDs per user's feed, at a few bytes each, is only a few KB per user — cheap to hold in a fast store for every active user, which is exactly why fan-out-on-write is attractive wherever it's affordable.

## High-level design

When a user posts, the write lands in a durable content store and is published as an event onto a pub/sub or message-queue backbone (see [pub/sub and event-driven architecture](/roadmaps/system-design/pub-sub-and-event-driven-architecture)). A fan-out worker tier consumes that event and, for accounts under the fan-out threshold, pushes the new post's ID into every follower's precomputed feed list, typically held in a fast cache or dedicated feed-storage tier. When a user opens their feed, the feed service reads that precomputed list directly — mostly a cache hit, mostly fast — merges in any posts from accounts handled via read-time fan-out (the celebrity exception, below), runs the combined candidate set through a ranking step, and returns the top results. Notifications follow the same event-driven backbone: an event (new like, new follower) is published, a notification worker decides who should be notified and through which channel (push, in-app badge, email), and a separate delivery service — reusing the chat system's connection-routing idea for in-app push, or a third-party push gateway for mobile — gets it to the user (see [caching and CDNs](/roadmaps/system-design/caching-and-cdns) for how the precomputed feed data itself gets served cheaply at read time).

## Key decisions

**1. Fan-out-on-write vs. fan-out-on-read — and the hybrid that production systems actually run.** Fan-out-on-write does the fan-out work once, at post time, so that every subsequent feed read is just a cheap lookup — great for the overwhelmingly common case of accounts with a normal follower count, where a single post generating a few hundred or few thousand feed-insertion events is trivial. But a celebrity account with 20 million followers turns one post into 20 million fan-out writes, which doesn't complete in a reasonable time and can flood the fan-out workers. Fan-out-on-read avoids this by never precomputing anything — every feed load pulls fresh from each followed account directly — but that makes *every* feed load pay a cost proportional to how many accounts a user follows, every single time, which is wasteful for the typical case. The standard resolution: fan out on write for accounts below a follower-count threshold, and treat accounts above it as a small, enumerable "merge in at read time" list that every follower's feed-read step checks and blends in on the fly — bounding the worst case without taxing the common one.

**2. Feed ranking, conceptually.** Assembling the candidate set of posts (via either fan-out path) is a separate problem from deciding what order to show them in. A purely reverse-chronological feed is simple and predictable but tends to under-serve users who don't check the app often (their feed is dominated by whatever posted most recently, not what they'd most want to see). A ranking step scores each candidate post on signals like recency, the poster's typical engagement with this viewer, and the post's overall engagement velocity, then orders by score rather than raw timestamp. This case study doesn't need to re-derive a ranking model — the key design point is just that ranking happens as a distinct stage *after* candidate assembly, operating over a bounded candidate set (not the entire platform's post history), which is what keeps it fast enough to run on every feed load.

**3. Reusing the same backbone for notifications, with a tighter latency bar.** Notifications are conceptually a feed with one item per event rather than one item per post, but they usually carry a stricter expectation of near-real-time delivery (a "someone liked your post" push feels broken if it arrives five minutes late, in a way a feed item doesn't). This pushes notification delivery toward the same real-time connection-routing approach used for chat, layered on top of the same pub/sub event backbone that drives feed fan-out — the event model is shared, but the delivery path's latency budget is tighter.

## Tradeoff and failure mode worth calling out

The sharpest tradeoff is that the **hybrid approach's threshold is itself a leaky abstraction**: an account that crosses the fan-out-on-write threshold mid-system-operation (going viral, suddenly gaining millions of followers) needs to transition from one fan-out strategy to the other without a visible glitch — followers shouldn't see a post twice, or miss one, during the switch. In practice this means the threshold check has to be re-evaluated continuously rather than fixed at account-creation time, and the fan-out worker tier needs headroom to handle an account's burst of *new* followers suddenly pushing it over the line, not just the steady-state traffic the threshold was sized for. It's a reminder that "celebrity accounts are the exception" is a statement about the *current* state of the graph, not a fixed property of any given account — and a feed system that only handles the exception case for accounts that were already large when the system launched will eventually get blindsided by one that wasn't.
