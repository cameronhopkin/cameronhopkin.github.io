---
title: "Day 025: The Arithmetic of Trust"
date: 2026-03-23T09:58:00-05:00
draft: false
tags: ["c", "switch", "binary-search", "overflow", "escaping", "knr"]
description: "Section 3.3 and 3.4. Every formula I trusted had a hole in it."
---

# Day 025: The Arithmetic of Trust

Chapter 3 this morning. I came in thinking control flow would be
familiar territory. It is. That is exactly the problem. Familiar
is where the traps hide.

## What I Did

Started with the binary search from section 3.3. K&R presents it
as a worked example before Exercise 3-1. Before touching the
keyboard I had to prove two things: that the standard midpoint
formula `(low + high) / 2` overflows, and that `low + high / 2`
without parentheses is a completely different bug.

They look similar. They are not.

`low + high / 2` is a precedence error. Division binds tighter
than addition, so the compiler reads it as `low + (high / 2)`.
No overflow. Just wrong math. Your midpoint drifts and the search
silently fails on valid data.

`(low + high) / 2` is the overflow. Force the addition first and
on a 32-bit signed int, once `low + high` crosses 2,147,483,647,
the sign bit flips. On Minion1 with `low = 1,500,000,000` and
`high = 1,600,000,000`, the result was `-597,483,648`. A negative
index. Entry 19 in the field guide, live on the machine.

The fix is `low + (high - low) / 2`. Same midpoint mathematically.
The intermediate value never escapes the safe range.

After that, Exercise 3-1: rewrite binsearch with one test inside the
loop. The standard version has two: a range check and an equality
check. The one-test version removes the early exit and turns the loop
into a pure narrowing engine. It runs to completion every time,
collapsing `low` and `high` until they meet, then checks equality
once at the end. The loop's job changed. So did the termination
condition.

Section 3.4 was the switch statement. Typed the digit-counting example
from page 59. The thing K&R says that most people skip: `case` labels
are labels in the strictest sense. The `default` can go anywhere in the
block. Convention puts it last. The compiler does not care.

Exercise 3-2 was the sharpest problem of the day: write `escape` and
`unescape` functions using `switch`. I built the first version, tested
it, and it passed everything I tried. Then I traced `"a\\b"` through
it by hand.

The `escape` function had no `case '\\'`. A literal backslash in the
input passed through unchanged. Then `unescape` saw that backslash
followed by `b`, treated it as an escape sequence, and hit `default`.
On simple inputs the round-trip looked clean. On a backslash followed
by `n`, it did not. The escaped form of a real newline and the escaped
form of a literal backslash plus `n` were identical. `unescape` had
no way to tell them apart.

The fix was one case: `case '\\': s[j++] = '\\'; s[j++] = '\\'; break;`

Round-trip confirmed on Minion1.

## The Questions That Came Up

### Why does `low < high` work as the termination condition in the one-test version when `low <= high` would cause an infinite loop?

With `high = mid` instead of `mid - 1`, if `low == high == mid` and
the element is present, the next iteration sets `high = mid` again.
Nothing moves. `low < high` breaks as soon as the window collapses to
one element. The final equality check outside the loop handles the
result. The termination condition changed because the loop's purpose
changed. It is not finding the element anymore. It is only narrowing.

### Is the one-test version faster?

Not universally. The early-exit version wins whenever it finds the
target before completing all iterations — on 8 elements, a hit on
the second comparison costs less than always running to completion.
The branch predictor argument is real, but only pays dividends when
the misprediction penalty (15-20 cycles on modern silicon) consistently
exceeds the cost of the extra comparison in the predictable path.
That requires large datasets in tight loops. Deterministic execution
is a hedge against branch misprediction at scale. It is not a
universal speed gain. The point of the exercise is the structural
shift: a loop that narrows vs. a loop that searches.

## The Feynman Test

Imagine a log system that stores user input by first "escaping" it
into a safe representation. Newlines become `\n`, tabs become `\t`.
The log reader later "unescapes" them back.

If the escaper never handles the backslash itself, the log now has
an ambiguity. The two-character sequence `\n` in a log entry could
mean "there was a real newline here" or "there was a backslash
followed by the letter n." The reader cannot tell. An attacker who
knows this types a literal backslash followed by `n` and the reader
decodes it as a newline — slipping content through a filter that was
only watching for the real thing.

The fix is mechanical. If `\` becomes `\\` in the escaped form, the
reader always knows that a lone `\` starts a sequence and `\\` is a
literal backslash. No ambiguity. Every possible input survives the
round-trip intact.

This is not a theoretical problem. It is the structural gap behind
SQL injection, shell injection, and most filter bypasses. The escape
character must escape itself. A sanitizer that does not handle its
own delimiter has not closed the boundary. It has just moved it.

## Hacker Connection

The delimiter collision in `escape` is the same structural gap that
underlies SQL injection. A query builder that escapes single quotes
but not backslashes can be bypassed with `\'`, which the escaper
passes through, causing the database to read the quote as data that
closes the string. The escaper believed it was protecting. It was not.

The midpoint overflow has a documented history. The Java standard
library carried the `(low + high) >> 1` form of this bug for decades
before it was reported publicly. Same overflow, expressed with a right
shift. Same fix: subtract before you add.

Notebook entries added today: 23 (Midpoint Sum Overflow, CWE-190),
24 (Precedence-Induced Range Shift, CWE-682), 25 (Fall-Through
Intuition Gap, CWE-484), 26 (Loop-Increment Bypass, CWE-670),
27 (Delimiter Collision, CWE-74).

## What Is Next

Sections 3.5 through 3.7: loops in full, break and continue, goto.
The `shellsort` example in 3.5 uses a nested loop with a
range-expansion pattern I have not worked through yet. Exercise 3-3
(`expand`) is the next terminal target.

Overnight question: `goto` exists in C. K&R will describe it. What is
the one legitimate use case K&R acknowledges, and why does that use
case disappear in languages with structured exception handling? Reason
it through before tomorrow.

---

*Day 25 of 365. Every formula I trusted had a hole in it. The machine
never lied. I just hadn't asked the right questions yet.*
