---
title: "Day 19: The Mask"
date: 2026-03-11T00:00:00-05:00
draft: false
tags: ["c", "bitwise", "hex", "vulnerability", "sign-extension"]
description: "Section 2.7, bitwise operators, and why the same 8 bits mean different things depending on who's reading them."
---

# Day 19: The Mask

We fell behind. No elaborate explanation. Life happened and the streak broke.
Day 19. Back on the machine.

The overnight question from Day 18 was sitting there: what does `&` do to two
integers, and where does it show up in security work? I came in with an answer
that confused `&` and `&&`. The distinction matters. One operates on truth.
The other operates on bits. Section 2.7 is about the one that operates on bits.

## What I Did

Opened K&R at section 2.7. Bitwise operators: `&`, `|`, `^`, `~`, `<<`, `>>`.
The chapter is dense and short. Every operator has a specific mechanical
behavior and a specific set of uses.

The overnight question forced me to sort out the `&` versus `&&` confusion
before touching the section. Logical AND returns 0 or 1 based on whether both
sides are non-zero. Bitwise AND compares each bit position independently.
1 AND 1 gives 1. Everything else gives 0. The result is an integer of the same
width as the operands, not a truth value.

From there we worked through where bitwise AND actually shows up. The answer
is Linux file permissions. When the kernel checks whether a user can read a
file, it performs a bitwise AND between the file's permission mode (a 12-bit
integer stored in the inode) and a bitmask representing the specific permission
being checked. Non-zero means access granted. Zero means denied. The mask
zeros out everything it does not care about and isolates the one bit that
matters. That is the masking pattern: extract a signal from a larger value
without disturbing the rest.

Then the session surfaced something I had not planned on. Reading through the
type conversion material alongside 2.7, I worked through what happens when a
`char` with the high bit set is used as an array index on a signed-char
platform. `0xFF` on Minion1 is `-1`. When the array indexing operator promotes
it to `int` for pointer arithmetic, it sign-extends. `a[c]` becomes
`*(a + (-1))`. The CPU computes an address one byte before the start of the
array. No crash, no warning, no bounds violation flagged. The machine does
exactly what the standard says and lands in memory it was never meant to touch.

Entry 9 in the notebook.

Exercise 2-3 was the applied work: write `htoi(s)`, a function that converts a
hexadecimal string to an integer, handling an optional `0x` or `0X` prefix and
the full range of valid hex digits. Three character ranges, three arithmetic
expressions:

```c
digit = c - '0';          /* numeric:    maps '0'-'9' to 0-9   */
digit = c - 'a' + 10;    /* lowercase:  maps 'a'-'f' to 10-15 */
digit = c - 'A' + 10;    /* uppercase:  maps 'A'-'F' to 10-15 */
```

The accumulation loop shifts the running total left one hex digit and adds
the new value: `n = 16 * n + digit`. `htoi("0x1A")` returned 26. `htoi("fff")`
returned 4095. Both correct on the first compile.

Then I investigated whether the `char` sign-extension issue applied to `htoi`.
It does not. The function compares `s[i]` against bounded ranges. A value of
`-1` fails every comparison and hits the `break`. A `unsigned char` cast
produces 255, which also fails every comparison and hits the `break`. Same
behavior, different path. The cast is not wrong as defensive hygiene, but it
does not change the outcome here. I wrote the fix, proved it was unnecessary,
and left the original. That is what the analysis showed.

## The Questions That Came Up

### Why does `&` produce an integer instead of a boolean?

Because it is not a boolean operation. It is a parallel operation on every bit
in the register simultaneously. The result has the same width as the operands
because every bit position has its own independent output. The integer result
encodes the full pattern of which bits were set in both operands.

### What does `htoi("0x")` return?

Zero. The prefix is consumed, the loop finds no valid hex digits, `n` is never
updated, and the function returns its initial value. Mathematically safe.
Semantically wrong. In a production parser, the caller cannot distinguish
between a genuine zero and a malformed input.

## The Feynman Test

Imagine a building with 12 light switches, each one controlling a specific
door in the building. The pattern of which switches are on or off is stored
as a 12-digit binary number.

When someone wants to open door 7, the building's security system does not
read all 12 switches and make a list. It uses a mask: a number with only the
bit for door 7 set. It ANDs the mask against the stored pattern. If the result
is non-zero, door 7's switch is on and access is granted. If the result is
zero, the switch is off and access is denied.

That is a Linux file permission check. The file's permission mode is the
building. The bitmask is the question. The AND is the answer.

The security risk: if the number representing who is asking ever becomes
negative due to a type conversion error, the mask might check a completely
different switch, or one that does not exist at all.

## Hacker Connection

Two vulnerability classes from today.

The first is the masking pattern's inverse: flag manipulation. Any system that
stores multiple boolean states in a single integer and checks them with bitwise
AND can be attacked if an integer overflow or truncation changes which bits
are set. Access control lists, capability flags, permission bitmasks. Get the
integer wrong and the AND check tells the kernel the wrong answer.

The second is the signed char array index class added to the notebook today.
It appears in real software anywhere `char` indexes into a lookup table. The
classic instance is character classification tables in older C libraries. A
function that looks up whether a character is alphanumeric by using the
character value as an index into a 128-entry table is safe for ASCII. Feed it
a byte with the high bit set on a signed-char platform and the index goes
negative. The access lands before the table. Whether that is an information
disclosure or a write primitive depends on what surrounds the table in memory.

The pattern connecting both: type assumptions are silent. The machine does
the arithmetic you told it to do. It does not know what you meant.

## What Is Next

Continuing through Chapter 2. Sections 2.8 through 2.12 cover increment and
decrement operators, bitwise and assignment operators, conditional expressions,
precedence and order of evaluation. The precedence table is where a lot of
real bugs live. Reading it carefully is not optional.

Overnight question: C has both `i++` and `++i`. They both increment `i` by 1.
What is the difference between them, and can you think of a situation where
using one instead of the other would produce a different result?

---

*Day 19 of 365. The gap is closed. The work continues.*
