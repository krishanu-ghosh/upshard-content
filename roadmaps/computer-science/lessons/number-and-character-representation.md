---
title: "How Numbers and Text Become Bits"
description: Floating-point representation, big- vs little-endian byte order, and how ASCII became Unicode and UTF-8.
roadmap: computer-science
node: number-and-character-representation
group: representation-and-caching
video:
  provider: youtube
  id: "MijmeoH9LT4"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 570
level: beginner
prerequisites:
  - bitwise-operators-and-manipulation
takeaways:
  - "A floating-point number is stored as a sign bit, an exponent, and a mantissa (fraction), the same scheme as scientific notation but in binary, which is why it can represent a huge range of magnitudes with a fixed number of bits."
  - "Floating point can't represent most decimal fractions exactly in binary, which is why 0.1 + 0.2 produces something like 0.30000000000000004 instead of 0.3 — the error is in the representation, not the arithmetic."
  - "Endianness is the order multi-byte values are stored in memory: big-endian stores the most significant byte first (like reading a number left to right), little-endian stores the least significant byte first, and the two are incompatible without translation."
  - "ASCII assigns each English letter, digit, and common symbol a 7-bit number (0-127), which was enough for English text but had no room for accented letters, other scripts, or emoji."
  - "Unicode assigns a unique \"code point\" number to essentially every character in every writing system, and UTF-8 is the encoding that turns those numbers into bytes — using 1 byte for ASCII characters and up to 4 for everything else, which keeps it backward-compatible with ASCII."
check_questions:
  - q: Why does `0.1 + 0.2` not equal exactly `0.3` in most programming languages?
    a: Binary floating point can only exactly represent fractions whose denominator is a power of two, and 0.1 and 0.2 aren't among them, so both are stored as the closest approximation, and adding two approximations compounds the tiny error.
  - q: What's the practical consequence of big-endian and little-endian systems disagreeing on byte order?
    a: A multi-byte value written to a file or sent over a network by a little-endian system will be misread by a big-endian system (and vice versa) unless one side converts it, which is why network protocols fix a standard byte order ("network byte order," which is big-endian) that every system converts to and from.
  - q: Why is UTF-8 described as "backward-compatible" with ASCII?
    a: Every ASCII character (code points 0-127) is encoded in UTF-8 as a single byte with the exact same bit pattern as ASCII uses, so any valid ASCII text is already valid UTF-8 text with no conversion needed.
deeper:
  - label: "Floating Point Numbers (Part1: Fp vs Fixed) - Computerphile"
    url: "https://www.youtube.com/watch?v=f4ekifyijIg"
  - label: "Endianness Explained With an Egg - Computerphile"
    url: "https://www.youtube.com/watch?v=NcaiHcBvDR4"
related_formulas: []
---


## Everything is bits — representation is the agreement on what they mean

A sequence of bits has no inherent meaning. The same 32 bits could be a small integer, a floating-point number, four ASCII characters, or a pixel's color, depending entirely on the convention the reading program agrees to use. This lesson covers three of the most consequential conventions: how fractional numbers are stored, what order multi-byte values are written in, and how text is turned into bytes.

## Floating point: scientific notation in binary

Integers are straightforward to store as binary — convert the number to base 2 and use enough bits. Fractional numbers are trickier, because you need to cover both very large and very small magnitudes with a fixed number of bits. The IEEE 754 standard does this the same way scientific notation does: split the number into a **sign** (positive or negative), an **exponent** (how large or small the magnitude is), and a **mantissa** (the significant digits), roughly `(-1)^sign × mantissa × 2^exponent`. A 32-bit "single precision" float spends 1 bit on the sign, 8 on the exponent, and 23 on the mantissa.

The catch: binary fractions can only exactly represent values that are sums of powers of two (like 0.5, 0.25, 0.125). The decimal value 0.1 is not one of those — in binary it's a repeating fraction, just like 1/3 is a repeating decimal in base 10. So 0.1 gets stored as the *closest representable value*, which is very slightly off. Add that tiny error to the tiny error in 0.2, and the result is 0.30000000000000004 rather than 0.3. This isn't a bug in any particular language; it's a direct consequence of representing base-10 fractions in base 2, and it's why financial software typically avoids floating point for currency and uses fixed-point or integer-cents representations instead.

## Endianness: which end of the number comes first

A single byte has no ordering question — it's 8 bits, read the same way everywhere. But most values (a 32-bit integer, for instance) span multiple bytes, and there are two sensible ways to lay them out in memory. **Big-endian** stores the most significant byte at the lowest memory address — the same order you'd write the number on paper, most-significant-digit first. **Little-endian** stores the least significant byte first. The number 0x12345678 would be stored as the byte sequence `12 34 56 78` on a big-endian system but `78 56 34 12` on a little-endian one (x86 and ARM in their common configurations are little-endian; many network protocols are big-endian).

Neither choice is "more correct" — they're just incompatible conventions. The practical consequence shows up whenever raw bytes move between systems: a file format or network protocol has to specify its byte order explicitly (network protocols conventionally use big-endian, historically called "network byte order"), and code that reads multi-byte values from a file or socket has to know which convention was used to write them, or it will reconstruct the wrong number entirely.

## From ASCII to Unicode: representing text

Text has the same underlying problem as numbers — bits need an agreed-upon meaning. **ASCII**, from the 1960s, solved it for English by assigning each letter, digit, and punctuation mark a number from 0 to 127, fitting in 7 bits (one byte with room to spare). `A` is 65, `a` is 97, the digit `0` is 48. This worked fine as long as the only text that mattered was unaccented English — it had no code points left for é, ñ, Chinese characters, Cyrillic, or emoji.

**Unicode** generalizes the idea: it assigns a unique number (a "code point") to essentially every character in every writing system in use, plus symbols, emoji, and more — over a million possible code points, though far fewer are currently assigned. Unicode itself only defines *which number means which character*; it doesn't say how to store that number as bytes. That's the job of an encoding, and **UTF-8** is the one that won. UTF-8 uses a variable number of bytes per character: 1 byte for code points 0-127 (exactly matching ASCII, bit for bit), up to 4 bytes for higher code points like emoji or less common scripts. Because ASCII text maps byte-for-byte onto UTF-8, decades of ASCII-only software and files remained valid without any conversion — a major reason UTF-8 displaced earlier fixed-width Unicode encodings as the web's default.
