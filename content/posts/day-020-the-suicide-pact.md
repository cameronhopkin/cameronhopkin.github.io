---
title: "Day 020: The C String Contract Is a Suicide Pact"
date: 2026-03-16T12:00:00-05:00
draft: false
tags: ["c", "knr", "increment", "strcat", "buffer-overflow", "undefined-behavior"]
description: "K&R 2.8, the strcat design flaw, four new notebook entries, and the moment I realized safety in C is often just geographic coincidence."
---

# Day 020: The C String Contract Is a Suicide Pact

The C string contract goes like this: you give the function a pointer, the function trusts you completely, and when you are wrong, you both go down. No negotiation. No error return. No last-second check. `strcat` does not ask how much space you have. It assumes you know. If you are wrong, it keeps writing until something breaks. That is not a bug. That is the design.

## What I Did

Came back from the weekend. Day 19 closed on the signed char vulnerability. Day 20 opened on the overnight question: what is the difference between `i++` and `++i`.

The answer turned out to be a thread that unraveled most of the session. Post-increment returns the current value and increments after. Pre-increment increments first and returns the updated value. In a standalone statement, the compiler emits identical code. The distinction only matters when the result is used in an expression. That led to sequence point rules, which led to `a[i] = i++`, which is undefined behavior because `i` is both modified and used as an index with no intervening sequence point. The compiler can legally do whatever it wants with that line. Entry 10 went into the notebook.

Section 2.8 followed. K&R uses two examples to show the operators in context: `squeeze`, which filters characters from a string, and `strcat`, which concatenates two strings. Both use post-increment write-head logic, both do something subtle, and both unraveled into notebook entries.

The `squeeze` exercise became Exercise 2-4. The extension: modify `squeeze` to delete all characters in `s1` that appear anywhere in `s2`, not just a single character. The fix is a nested loop with a flag. The write-head is `k`, tracking where the next surviving character lands. Using `s1[k++] = s1[i]` means "write here, then advance." Swapping it to `s1[++k] = s1[i]` skips index 0, leaves the original first character in place as "ghost" data, and writes the null terminator one byte past the buffer boundary. If that string held a password or a sensitive token, the squeeze function just left the most important piece behind. Entry 11.

Exercise 2-5 was cleaner: `any(s1, s2)` returns the first index in `s1` where any character from `s2` appears, or `-1` if none. The nested loop with early return writes itself. What took longer was the question that followed: is `-1` here the same pattern as EOF in Entry 1? The answer is no, but the distinction is unstable. In `any`, `-1` is safe as a "not found" signal because array indices start at zero. The value is out-of-band by geography. But if the function returned relative offsets instead of absolute indices, `-1` could mean both "one step behind you" and "nothing found." The return channel carries two types of messages and the receiver has to know which one arrived. Entry 12.

The `strcat` design flaw landed last. The signature is `void strcat(char s[], char t[])`. No capacity parameter. The function accepts a destination buffer and a source but has no way to know how large the destination is. It trusts the null terminator of `s` to find the end, then trusts the null terminator of `t` to stop copying. If the programmer miscalculated the space, the function cannot detect it. Safety is outsourced entirely to the caller. Entry 13.

Four entries today. All written by me.

## The Questions That Came Up

### Is `(s[i++] = t[j++]) != '\0'` undefined behavior?

Looks like it might be. Two post-increments, an assignment, all inside a single expression. Applied the Entry 10 test: identify every modified object, identify every read of that object, check whether any modification and read of the same object are unsequenced.

Result: not UB. `i` is read to compute the destination index, and the increment is a side effect of that sub-expression, sequenced after the value computation. Same for `j`. The two objects are distinct. No single object is both modified and read in an unsequenced way. The code is well-defined.

The trap was surface pattern matching. Two `++` operators, assignment inside a condition — looks dangerous. The mechanics say otherwise.

### Is the `-1` in `any()` the same as EOF?

No, but only by coincidence of geography. EOF is dangerous because it can collide with valid data when the wrong type is used. The `-1` in `any()` is safe because indices are non-negative by definition. Move to relative offsets, circular buffers, or signed distances and the collision becomes real. Safety was never a guarantee. It was a byproduct of where arrays happen to start.

## The Feynman Test

Imagine a moving company that will pack and ship everything in your house, but they never ask how big the truck is. You tell them where to start. They trust you pre-arranged enough space. If the truck fills up, they keep loading anyway because nobody told them to stop.

That is `strcat`. The function knows where `s` starts, finds where it ends by walking until it hits a null byte, then copies characters from `t` until it hits another null byte. It has no knowledge of how large the buffer is. If the programmer allocated 20 bytes but the combined strings need 30, `strcat` writes the extra 10 bytes into whatever memory follows the buffer. Stack variables. Return addresses. Whatever is there.

The reason this matters for security: those 10 extra bytes are attacker-controlled. If an attacker can feed a long enough string into a `strcat` call, they control what gets written past the buffer boundary. Control what gets written to a return address and you control where the program goes next. That is a buffer overflow. That is how the Morris Worm worked in 1988. That is how buffer overflows still work today.

The fix is not hard: `strncat` takes a third argument — the maximum number of characters to append. The information the caller has always had gets passed to the function. The function stops when it hits the limit. The design hole closes. The original function cannot be unfixed; it is in the standard. Every new C program written with `strcat` today is making the same bet the original programmers made: that the caller got the math right.

## Hacker Connection

The `strcat` design flaw is CWE-120: Classic Buffer Overflow, also called the "buffer copy without checking size of input." It is one of the oldest and most documented vulnerability classes in software security.

The Morris Worm of 1988 used a `gets` call — the same design philosophy, no bounds check, trust the input to terminate — to achieve remote code execution on VAX and Sun machines running BSD Unix. That was 38 years ago. The C standard library still ships `strcat` and `gets` (deprecated in C11, removed in C17, but the damage was done).

The notebook now has 13 entries. The thread connecting them is becoming visible: Entry 1 (sentinel collision), Entry 9 (signed char index), Entry 12 (sentinel-to-offset promotion), and Entry 13 (unbounded sentinel trust) are all variations of the same root cause — a single return channel or a single value carrying two types of meaning, with no mechanism to tell them apart. The machine cannot tell. The programmer has to. When the programmer gets it wrong, the machine keeps going.

## What Is Next

Section 2.9: bitwise operators (a brief revisit before the chapter closes out). Then 2.10 and 2.11 — assignment operators and expressions, the conditional expression. Chapter 2 is close to done.

Overnight question: K&R uses the conditional expression `(n > 0) ? f : -f` as an example. What is the difference between writing that as a conditional expression versus writing it as an if-else? Is there a case where the compiler generates different code for each? Is there a security case where the two behave differently?

Hacker track next: the notebook has 13 entries and the patterns are starting to reference each other. Before Chapter 3, a review pass — map the entries that share a root cause. The thread is there. Name it explicitly before Chapter 3 introduces new ones.

---

*Day 20 of 365. The contract was always a bet. You just learned what you were betting.*
