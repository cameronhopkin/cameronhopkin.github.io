---
title: "Day 022 The High-Voltage Basement"
date: 2026-03-18T00:00:00-05:00
draft: false
tags: ["c", "bitwise", "undefined-behavior", "exploit-dev", "knr"]
description: "Chapter 2 closes. The compiler caught my UB live. Six operators, two exercises, sixteen notebook entries."
---

# Day 022: The High-Voltage Basement

Chapter 2 is closed. Not filed away. Closed the way a circuit closes when current finally finds its path.

Today was K&R section 2.9 through 2.11. Bitwise operators, assignment operators, conditional expressions. On paper that sounds like a syntax lesson. What it actually was: a guided tour of the places where the compiler is allowed to let you destroy yourself, and the handful of words you can add to your code to make it stop.

## What I Did

Started the session by reasoning through all six bitwise operators before opening the book. AND, OR, XOR, left shift, right shift, bitwise NOT. I knew what they did in the abstract. The session forced me to prove it with bits on paper, decimal by decimal, before writing a line of code.

Then I wrote `invert()` for Exercise 2-7. The function takes a value, a position, and a width, and flips exactly those bits using XOR. The plan was clean: build a window of ones with `~(~0 << n)`, position it with a left shift, apply it with XOR. I typed it, compiled it, and got an immediate error.

```
ex2-07.c:8:18: error: left shift of negative value [-Werror=shift-negative-value]
```

The compiler refused. `-Werror` promoted the warning to a hard stop. The problem was `~0`. In C, the literal `0` is a signed int. Applying `~` to it produces `-1`. Shifting a negative signed integer left is undefined behavior under the standard. The compiler knew. I had literally just written Entry 15 into the notebook and then written it into my code thirty minutes later.

The fix is one character. Change `~0` to `~0U`. The `U` suffix makes the literal unsigned. No sign bit. The shift is now well-defined. The compiler compiled.

Exercise 2-10 followed: rewrite `lower()` using a conditional expression. Simpler, but the same precision discipline. Hand-calculate first, then code, then verify. `'G'` is ASCII 71. Add 32. Get 103. Get `'g'`. The test passed.

Sixteen notebook entries. Chapter 2 done.

## The Questions That Came Up

### Why does `~0` produce negative one?

In two's complement, zero is all zero bits. `~` flips every bit. All ones in a signed integer is the bit pattern for -1. This is not a quirk. It is the definition. `~0` producing -1 is the expected behavior of the type system working exactly as designed. The problem is that left-shifting a negative value is undefined. The type system is consistent. The shift rules are not.

### Is the ternary operator actually constant-time?

No. The C standard does not mandate how an expression translates to assembly. A compiler can generate a `CMOV` (conditional move, no branch) or a `JZ/JNZ` pair (conditional branch, pipeline-visible) for the same ternary expression depending on architecture, optimization level, and compiler version. If you need constant-time execution in cryptographic code, you do not use a ternary. You use bitwise arithmetic masks built from unsigned types with explicit logic. The ternary is a convenience. It is not a security primitive.

### What does "implementation-defined" actually mean for right shift?

It means the C standard permits two behaviors and does not choose between them. Logical right shift fills vacated bits with zeros. Arithmetic right shift fills them with copies of the sign bit. Most modern x86 and ARM implementations use arithmetic shift for signed types. Most. If you write code that depends on arithmetic shift behavior for signed integers and move it to a platform that uses logical shift, a bounds check that was passing negative values will start passing values near `INT_MAX`. The fix is always the same: cast to unsigned before shifting if you care about the bit pattern.

## The Feynman Test

Imagine a row of light switches. Each switch is either on or off, nothing else.

In C, if you just write the number zero, the machine assumes one of those switches is a "sign switch." It is reserved. It tells the machine whether the whole number is positive or negative. When you flip every switch with `~`, you are flipping the sign switch too. On a signed number, shifting after that flip is undefined because the machine cannot decide what moving a negative number to the left means. Different architectures make different choices. The standard gave up and said "your problem."

The `U` in `~0U` removes the reservation. There is no sign switch. Every switch is just data. Flip them all, shift freely. The machine does not panic because there is no special switch to misinterpret.

That is what undefined behavior is. Not a crash. Not a warning. A place where the standard stepped back and said the machine can do whatever it wants. Your job is to not stand there.

## Hacker Connection

The `-Werror` catch today is the same mechanism that would have stopped a real vulnerability. Left-shift sign overflow is how integer values in security-critical code silently change sign. A permissions mask built from a shifted negative value is a mask that does not do what you think it does. The wrong bits get set. The wrong access gets permitted.

The bitwise mask pattern from today has a direct cousin in exploit development and cryptographic implementation. To select between two values based on a secret bit without creating a timing channel, production cryptographic code builds a mask like this:

```c
unsigned int s_norm = !!s;        /* normalize to 0 or 1 */
unsigned int mask = -(s_norm);    /* 0x00000000 or 0xFFFFFFFF */
result = (a & mask) | (b & ~mask);
```

No branch. No timing signal. The CPU executes the same instructions regardless of `s`. The key requirement: `s` must be normalized to exactly 0 or 1 first, and all types must be unsigned. One deviation and the mask breaks silently. This is the same discipline we applied to `~0U` today, scaled up to a cryptographic primitive.

The notebook now has sixteen entries. The three recurring roots are visible: ambiguous sentinels, implementation gambles, and side-effect races. Every new entry is a variation on one of those three. Knowing the roots makes the variants recognizable on first contact.

## What Is Next

Chapter 3. Control flow. If-else, switch, while, for, do-while, break, continue, goto.

Before opening the chapter: the notebook has sixteen entries and the three root causes are named. The hacker track task before Chapter 3 begins is a review pass. Map every entry to its root cause. The thread is there. Name it explicitly so that when Chapter 3 introduces new patterns, the roots are already visible.

Overnight question: `goto` exists in C. K&R uses it exactly once in the book, for a specific purpose. What do you think that purpose is, and why is `goto` considered dangerous enough that most modern coding standards ban it outright?

---

*Day 22 of 365. The compiler is not your enemy. It is the first line of defense you kept trying to walk around.*
