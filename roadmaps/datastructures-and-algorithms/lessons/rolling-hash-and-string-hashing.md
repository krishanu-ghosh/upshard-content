---
title: "Rolling hash and string hashing"
description: Updating a sliding-window string hash in O(1) per shift, and how it powers Rabin-Karp-style pattern matching.
roadmap: datastructures-and-algorithms
node: rolling-hash-and-string-hashing
group: more-patterns-and-tricks
video:
  provider: youtube
  id: "qQ8vS2btsxI"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 1430
level: intermediate
prerequisites: [string-matching-and-suffix-arrays]
takeaways:
  - "A rolling hash treats a string window as a number in some base b: hash(s[0..m-1]) = s[0]*b^(m-1) + s[1]*b^(m-2) + ... + s[m-1]*b^0, all computed modulo a large prime p to keep values bounded and reduce collisions."
  - "Sliding the window by one character updates the hash in O(1) rather than O(m): subtract the outgoing character's weighted contribution (old_char * b^(m-1)), multiply the remainder by b to shift every remaining digit up one place, then add the incoming character's contribution — new_hash = (old_hash - old_char*b^(m-1)) * b + new_char, all mod p."
  - "Using a prime modulus p (and ideally a base b coprime to p, or just a well-chosen prime base) spreads hash values evenly across the output range and minimizes collisions — a poor modulus choice (like a power of 2) can create systematic clustering that defeats the hash's purpose."
  - "Rabin-Karp uses this rolling hash to find pattern matches in O(n+m) expected time: compute the pattern's hash once, then slide a same-length window across the text updating the hash in O(1) per shift, only falling back to an O(m) character-by-character comparison when hashes match (to rule out hash collisions, since a hash match doesn't guarantee string equality)."
  - "The same rolling-hash trick extends to multi-pattern matching (hash all patterns into a set, then check each window's hash against the set in O(1) average) and to duplicate-substring problems (hash every length-L window of a string and look for repeated hash values, which is the core of the 'longest duplicate substring' binary-search-on-length technique)."
  - "A single modulus is vulnerable to adversarial inputs engineered to produce hash collisions; production-grade implementations often use two independent (base, modulus) pairs and combine both hashes, which makes deliberately engineered collisions astronomically unlikely without meaningfully increasing the per-shift cost."
check_questions:
  - q: "Walk through why the rolling hash update formula (old_hash - old_char*b^(m-1)) * b + new_char correctly produces the hash of the shifted window, rather than some unrelated value?"
    a: "The old hash represents the window as a base-b number with the outgoing character as the most significant digit. Subtracting old_char*b^(m-1) removes exactly that digit's contribution, leaving the remaining m-1 characters' weighted sum. Multiplying by b shifts every remaining digit one place to the left (equivalent to how multiplying a base-10 number by 10 shifts its digits), which is exactly the effect of the window moving one position to the right. Adding new_char then places the incoming character correctly as the new least-significant digit — reconstructing exactly the hash you'd get by recomputing the new window's hash from scratch, but in O(1) instead of O(m)."
  - q: "Why does Rabin-Karp still need a character-by-character verification step after a hash match, instead of trusting the hash match as proof of an actual substring match?"
    a: "A hash function maps a much larger space of possible strings down to a bounded range of integers modulo p, so by the pigeonhole principle, distinct strings can and sometimes do produce the same hash value (a collision). A hash match is therefore only strong evidence of equality, not proof; verifying the actual characters guards against the (hopefully rare, with a good modulus) case where two genuinely different substrings happen to hash identically, which would otherwise produce a false positive match."
  - q: "Why does using a prime number as the modulus matter for rolling hash quality, rather than, say, a round number like 1,000,000?"
    a: "A prime modulus, especially combined with a base that isn't a factor of it, tends to distribute hash values much more uniformly across the output range, because there are no small common factors between the base's powers and the modulus to create repeating patterns or clustering. A composite or power-of-two modulus can interact badly with how the base's powers cycle, causing certain classes of strings to systematically collide far more often than chance would predict, which degrades Rabin-Karp's expected O(n+m) performance toward its O(n*m) worst case."
deeper:
  - label: "Rabin Karp Substring Search Pattern Matching — Tushar Roy, Coding Made Simple"
    url: "https://www.youtube.com/watch?v=H4VrKHVG5qI"
related_formulas: []
---

## Why hash a sliding window at all

Many string problems need to repeatedly ask "does this length-m window match some
target?" or "have I seen this exact substring before?" across every window as you slide
across a longer string. Comparing characters directly at every position costs O(m) per
window, so scanning all `n-m+1` windows is O(n*m). A rolling hash collapses the "does
this window match" check to an O(1) integer comparison per shift, after an O(m) cost to
hash the very first window — turning the whole scan into O(n+m) expected time.

## Treating a string as a number

Pick a base `b` (commonly a prime like 31 or 131, or the alphabet size) and a large
prime modulus `p`. Treat each character's code as a digit, and hash a window the way you
would read a number in base `b`:

```
hash(s[0..m-1]) = (s[0]*b^(m-1) + s[1]*b^(m-2) + ... + s[m-2]*b + s[m-1]) mod p
```

This is exactly analogous to how `"present day digits 1 9 3"` form the base-10 number
193 — position matters, and the leftmost character carries the most weight (`b^(m-1)`).

## The O(1) shift: subtract, multiply, add

To slide the window one character to the right — dropping `s[0]` and picking up the new
trailing character `s[m]` — don't recompute the sum from scratch. Instead:

```
new_hash = ((old_hash - s[0]*b^(m-1)) * b + s[m]) mod p
```

Three steps, each O(1) given `b^(m-1) mod p` precomputed once:

1. **Subtract** the outgoing character's weighted contribution, `s[0]*b^(m-1)`, removing
   its digit from the number entirely.
2. **Multiply** the remainder by `b`. This is the same effect as multiplying `193` by
   `10` to get `1930` — every remaining digit shifts one place to the left, making room
   for a new least-significant digit.
3. **Add** the incoming character `s[m]` as the new last digit.

(In modular arithmetic, subtraction can go negative, so implementations typically add
`p` before taking `mod p` again to keep the result non-negative.)

## Worked example

Let base `b = 10`, modulus `p = 1000003` (a prime, chosen just large enough to keep this
example's numbers small), and treat digits as their numeric values directly for clarity.
Window size `m = 3`, text `"1 9 3 7"` (digits 1, 9, 3, 7).

Initial window `"193"`: `hash = 1*10^2 + 9*10^1 + 3*10^0 = 100 + 90 + 3 = 193`.

Slide to `"937"` (drop `1`, add `7`): precompute `b^(m-1) = 10^2 = 100`.

```
new_hash = (193 - 1*100) * 10 + 7
         = (193 - 100) * 10 + 7
         = 93 * 10 + 7
         = 937
```

Matches recomputing `9*100 + 3*10 + 7*1 = 937` directly — but the rolling update did it
in three arithmetic operations instead of re-reading all three characters. With a real
modulus in play, every step above would also take `mod p`, keeping values bounded
regardless of how long the string or how large the window gets.

## From rolling hash to Rabin-Karp

Rabin-Karp pattern matching hashes the pattern once, then slides a same-length window
across the text using the O(1) update above, comparing hashes at each position. A hash
match triggers a direct character comparison (since hashing is many-to-one, a collision
is possible — equal hashes don't guarantee equal strings) before declaring a true match.
With a well-chosen prime modulus, collisions are rare, so the verification step rarely
triggers, keeping the expected running time at O(n+m).

The same machinery extends naturally: hashing multiple patterns into a set turns
Rabin-Karp into an O(1)-average multi-pattern matcher (check each window's hash against
the whole set at once), and hashing every length-L window of a single string to find
repeated hash values is the core subroutine behind "longest duplicate substring"
algorithms, typically combined with binary search over the candidate length `L`.
