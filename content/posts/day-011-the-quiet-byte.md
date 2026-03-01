---
title: "Day 011: The Quiet Byte"
date: 2026-02-28T18:00:00-05:00
draft: false
tags: ["c", "strings", "null-terminator", "security", "buffer-overflow"]
description: "K&R 1.8 and 1.9. Call by value, character arrays, and the single byte that holds all of C together."
---

# Day 011: The Quiet Byte

K&R 1.8 is short. Call by value. C copies everything you pass
to a function. Everything except arrays. That one exception
changes the entire security landscape of the language.

## What I Did

Worked through sections 1.8 and 1.9. Section 1.8 covers how
arguments pass to functions. Every variable gets copied. The
function works on the copy. The caller never knows. Simple.
Safe. Until arrays show up.

When you pass an array, C passes the address of the first
element. Not a copy. The original. The function can reach into
the caller's memory and change whatever it wants. No guardrails.

Section 1.9 is where it gets real. Character arrays. The
longest-line program. Two functions: `getline` reads characters
into a buffer, `copy` moves one string into another. C does not
have strings. It has arrays of characters that end with `'\0'`.
That single byte is the only thing telling the program where the
data stops and the unknown begins.

## The Questions That Came Up

### What happens across multiple files?

K&R assumes the whole program lives in one file. That is a big
assumption. If `getline` and `copy` were in a separate file, the
compiler would not know their signatures when compiling `main`.
You need forward declarations or a header file. Without them, old
C would guess the function signature. Silently. Wrongly. The same
class of problem I hit on Day 1 when `main` had no return type
and clang refused to compile it.

### Is null really just zero?

I always thought a null pointer was something undefined. It is
not. `'\0'` is the integer value 0 stored in a char. A null
pointer is the address 0. Both zero. Different meanings. The
confusion between them has caused real bugs in real codebases.

### What about the overflow K&R admits to?

K&R says this directly: `getline` checks for overflow. `copy`
does not. Their reasoning is that the caller of `copy` already
knows how big the strings are. That sentence is the design
philosophy that launched an entire vulnerability class. The
programmer assumed the caller would behave. Attackers do not
behave.

## The Feynman Test

A C string is a row of characters in memory with a zero at the
end. That zero is how every function knows when to stop reading.
There is no length field. No boundary marker the hardware
enforces. Just a convention that says "when you see zero, stop."

If the string fills the array exactly and there is no room for
the zero, the program does not stop. It keeps reading into
whatever memory comes next. Maybe that memory happens to be zero
already. The program works fine. Ship it. Then the memory layout
changes on a different machine or a different day and the program
reads into something that is not zero. Now it is reading data it
was never supposed to see.

That is not a crash. That is a data leak. That is Heartbleed.

## Hacker Connection

On Day 10 I watched a buffer overflow in GDB. I saw data land
past the end of the array and corrupt the stack. Today I
understand why that happens at the language level. C does not
copy arrays when passing them to functions. Functions write
directly into the caller's memory. And the only thing marking
the boundary of a string is a single byte that the programmer
is responsible for placing correctly.

The `copy` function in K&R does not check if the destination is
big enough. K&R acknowledges this. Multiply that decision across
fifty years of C code and you get the buffer overflow as the
most exploited vulnerability class in the history of computing.
The null terminator is a gentleman's agreement. Exploitation
begins when someone stops being polite.

## What Is Next

Exercises 1-16, 1-17, and 1-18. These are the character array
exercises and they build directly on the longest-line program.
Then section 1.10 on scope and external variables. That is the
last section of Chapter 1. The end of the foundation.

---

*Day 11 of 365. The most dangerous byte is the one that isn't there.*
