---
title: "Day 016 The Type System Bites"
date: 2026-03-05T00:00:00-05:00
draft: false
tags: ["c", "knr", "chapter2", "types", "integer-overflow", "undefined-behavior"]
description: "Chapter 2 opens and the type system immediately starts lying to you."
---

# Day 016: The Type System Bites

Chapter 1 is done. Chapter 2 opened today with what looks like dry bookkeeping. Data types. Sizes. Qualifiers. I knew going in that this section maps directly to some of the worst vulnerability classes in software history. I was not wrong.

## What I Did

Started in sections 2.1 and 2.2. Read them, took notes, flagged what did not make sense. Then built exercise 2-1: print the ranges of every integer and float type, first using the constants from `limits.h` and `float.h`, then computing them directly using bit manipulation.

The computed section was the interesting part. The `~0` trick for unsigned max. Then right-shifting by one to derive the signed max. Then bitwise NOT to get the signed min. Two sections printing the same values two different ways. If they match, the program verified itself. They matched.

Along the way I ran into `abs(INT_MIN)` and watched it crash differently each time. That is undefined behavior. The mathematical result of `abs(INT_MIN)` is `2^31`. That number does not fit in a signed int. The signed range is not symmetric. One more negative value than positive. The C standard says this is your problem, not the compiler's.

Also corrected my mental model on `FLT_MIN`. It is not the most negative float. It is the smallest positive normalized float. The most negative float is `-FLT_MAX`. The naming is a trap.

## The Questions That Came Up

### Is the underscore naming rule hard and fast?

No. Convention backed by a real collision risk. The C standard reserves identifiers starting with underscore followed by uppercase, or double underscore, for the implementation. If your name collides with a library routine using the same convention, you get undefined behavior with no warning. Treat it as off limits unless you know exactly why you are going there.

### What is the 31 significant character limit about?

Early linkers had fixed-width symbol tables. 31 characters fit in 32 bytes including the null terminator. It is a storage constraint from the hardware era that got baked into the standard. Not design wisdom. Archaeology.

### Is modulo 2^n for unsigned arithmetic a choice?

It is a guarantee. The standard says unsigned arithmetic wraps. Signed integer overflow is undefined behavior. That distinction matters and I will come back to it.

## The Feynman Test

Integer types have a range. Signed int on a 32-bit system runs from roughly negative two billion to positive two billion. Unsigned int runs from zero to roughly four billion. Same 32 bits. Different range because one bit is reserved for sign in the signed version.

Here is the trap. The negative side has one more value than the positive side. Negative two billion and change has no positive equivalent in the same type. So if you write code that takes a number, assumes it might be negative, and calls `abs()` to make it safe, you have a bug. Send in the most negative value and the result overflows. The safe path explodes.

This is not theoretical. Length fields in network protocols get sign-checked this way. An attacker who knows the type sends the most negative value. The sign check passes. The size is now wrong. Memory operations follow. You know where that goes.

## Hacker Connection

Two patterns went into the notebook today.

Signed integer overflow is undefined behavior in C. The compiler is not required to produce any particular result. It can assume overflow never happens and optimize accordingly. This creates situations where security checks get compiled away entirely. A bounds check that relies on signed overflow wrapping predictably can be removed by an optimizing compiler because the standard says that path cannot exist. The check disappears in release builds and not in debug builds. The bug is invisible until it is not.

The INT_MIN absolute value bug is its own category. A program receives a value, negates it or takes its absolute value expecting a positive result, then uses that result for a size or index. Attacker sends INT_MIN. The result is still negative or garbage. The downstream memory operation uses it anyway. This pattern shows up in real CVEs in size calculation code.

Both root causes are the same: the programmer assumed the type's behavior matched their mental model. It did not.

## What Is Next

Section 2.3 stopped at character constants. That is where tomorrow starts.

The overnight question: a character constant like `'A'` is an `int` in C, not a `char`. Why would K&R make a character literal an integer type? What breaks if you assume it is a char?

Hacker track is still building. The vulnerability notebook is growing. Phase 2 exploitation work is on the horizon.

---

*Day 16 of 365. The type system looks boring until it hands an attacker the keys.*
