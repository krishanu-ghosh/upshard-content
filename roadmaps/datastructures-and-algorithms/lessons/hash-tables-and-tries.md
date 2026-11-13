---
title: "Hash Tables and Tries"
description: How hash tables turn a key into near-instant lookup, how collisions get resolved, and when a trie beats a hash table for strings.
roadmap: datastructures-and-algorithms
node: hash-tables-and-tries
group: core-data-structures
video:
  provider: youtube
  id: "shs0KM3wKv8"
  start: 0
  end: null
creator:
  name: HackerRank
  channel_url: https://www.youtube.com/@HackerrankOfficial
duration_seconds: 540
level: beginner
prerequisites:
  - arrays-linked-lists-stacks-and-queues
takeaways:
  - A hash table maps a key to an array index by running the key through a hash function, which is what turns "look up by key" into an O(1) average-case operation instead of a scan.
  - Two different keys can hash to the same index — a collision — and every hash table needs a strategy for it, typically chaining (a linked list per slot) or open addressing (probing for the next free slot).
  - A hash table's O(1) average case degrades toward O(n) worst case if the hash function clusters keys unevenly or the table gets too full, which is why resizing and load factor matter.
  - A trie stores strings one character per edge, so all strings sharing a prefix share the same path from the root — that shared structure is what makes prefix search and autocomplete fast.
  - A hash table tells you only whether a string is present; a trie can also tell you, in time proportional to the string's length, every stored string that starts with a given prefix.
check_questions:
  - q: Two keys, "cat" and "act", hash to the same array index. What just happened, and what are the two common ways to handle it?
    a: That's a hash collision — different keys mapping to the same slot, which is unavoidable once there are more possible keys than slots. Chaining resolves it by storing a small linked list (or similar structure) at that slot holding every key that landed there; open addressing resolves it by probing forward to the next empty slot according to a fixed rule and storing the key there instead.
  - q: Why does a hash table's average-case O(1) lookup not hold if the table is nearly full?
    a: As the table fills up, collisions become far more frequent — with chaining, each slot's list grows longer and has to be scanned linearly; with open addressing, each insert or lookup has to probe through more already-occupied slots before finding the right one or an empty one. Both push the effective cost from O(1) toward O(n), which is why implementations resize (rehash into a bigger table) once the load factor crosses a threshold.
  - q: You need to build autocomplete that suggests every word in a dictionary starting with the letters a user has typed so far. Why is a trie a better fit here than a hash table of all the words?
    a: A hash table can only tell you whether one exact string is present — it has no notion of "nearby" strings, so finding every word with a given prefix would mean scanning the whole table. A trie's structure means every word sharing that prefix sits in the same subtree, reachable by walking the prefix's characters from the root, so finding all of them costs only the length of the prefix plus however many matches there are.
related_formulas: []
---

## Turning a key into an address

A plain array gives O(1) access, but only if you already know the index. A **hash table** (also called a hash map or dictionary) makes that O(1) access available for arbitrary keys — strings, tuples, custom objects — by running the key through a **hash function** that converts it into an integer, then using that integer (reduced modulo the table's size) as an array index. Insert a key-value pair, and the table hashes the key, drops the value at that index, and the lookup for the same key later recomputes the same hash and goes straight to the same index. No scanning, no comparing against every stored key — just one function call and one array access, which is why hash table operations are described as O(1) on average.

The "on average" is doing real work in that sentence. A good hash function spreads keys roughly evenly across the table's slots, but it's still entirely possible — guaranteed, in fact, once you have more possible keys than slots — for two different keys to hash to the same index. That's a **collision**, and every hash table implementation needs a defined way to handle it.

## Two ways to handle a collision

**Chaining** keeps a small secondary structure, usually a linked list, at each slot. When a collision happens, the new key-value pair is just appended to that slot's list. A lookup hashes the key to find the right slot, then walks that slot's (hopefully short) list to find the matching key. This is simple to implement and degrades gracefully — even a slot with several collisions still works, just more slowly.

**Open addressing** takes a different approach: there's no secondary structure, and every key lives directly in the main array. When a collision occurs, the table **probes** — checks the next slot according to some fixed rule (the simplest being "just try the next index over," called linear probing) — until it finds an empty one, and puts the key there. A lookup follows the same probing sequence starting from the key's hashed index until it either finds the key or hits an empty slot, which means the key isn't present. Open addressing can be more cache-friendly than chaining, since it avoids following pointers to scattered linked-list nodes, but it's more sensitive to the table filling up, because probing sequences get longer as there are fewer empty slots to land on.

In both schemes, the table tracks its **load factor** — the ratio of stored items to total slots — and resizes (allocates a bigger array and reinserts everything with a new modulus) once that ratio crosses a threshold, typically around 0.7. Without resizing, a nearly-full table's collisions compound until performance approaches the O(n) of a plain linear scan, which defeats the entire point of hashing.

## Tries: when the key is a string and prefixes matter

A hash table treats a string key as an opaque blob — it hashes the whole thing and loses any notion of the string's internal structure. That's fine for exact-match lookups, but it means a hash table can't answer "what words start with these letters?" without checking every stored word.

A **trie** (pronounced "try," from re**trie**val) is built specifically around that structure. Each node represents one character, and a path from the root spells out a string one edge at a time. Crucially, strings that share a prefix share the same path through the tree up to where they diverge — "car," "cart," and "care" all share the root → c → a → r path, then split into separate branches. Each node that marks the end of a complete stored word is flagged, since a shorter stored word can be a prefix of a longer one.

That shared-prefix structure is exactly what powers autocomplete: given a prefix a user has typed, you walk the trie one character at a time along that prefix — which takes time proportional only to the prefix's length, not the size of the dictionary — and then explore every path below that point to collect all the words that continue it. A hash table of the same words has no equivalent operation; it would need to inspect every single key to find the ones that happen to start with the right letters.

## Choosing between them

Reach for a **hash table** whenever the question is "is this exact key present, and what's its value?" — it's the default for dictionaries, caches, and sets, and it's simpler to implement and usually faster in practice for pure exact-match workloads. Reach for a **trie** when the keys are strings and you care about prefix relationships between them: autocomplete, spell-checkers, IP routing tables (which match longest address prefixes), and dictionary-word games all lean on a trie's shared-prefix structure rather than treating each key as an isolated blob.
