---
title: "Day 004 What They Didn't Know"
date: 2026-02-21T18:00:00-05:00
draft: false
tags: ["C", "K&R", "preprocessor", "security"]
description: "K&R section 1.4. Symbolic constants. The preprocessor. And the realization that every layer was built by people who could not see what was coming."
---

# Day 004: What They Didn't Know

K&R section 1.4 is barely two pages. Symbolic constants. Replace
magic numbers with named values using `#define`. Simple enough. But
the rabbit hole under those two pages went deep today.

## What I Did

Took the for loop from yesterday and replaced the hard coded numbers
with `#define` constants. `LOWER`, `UPPER`, `STEP`. All caps. No
semicolons. The program behaves the same but now a human reading it
knows what those numbers mean.

The code change was small. The reading around it was not.

I spent most of the hour researching what `#define` actually does.
It is not C. It is a directive to the preprocessor, a separate pass
that runs before the compiler ever sees your code. The preprocessor
does blind text replacement. No type checking. No scoping. No
awareness of C syntax at all. It just copies and pastes.

That distinction matters. `#define` and `const` look like they do
the same thing but they live in different worlds. `#define` is
resolved before compilation. `const` is resolved during or after.
In C, `const int` is not a true compile time constant. You cannot
use it for array sizes or case labels. `#define` works there because
the value is already substituted before the compiler runs.

## The Questions That Came Up

### Why no semicolon after `#define`?

Because the preprocessor copies everything after the name. Add a
semicolon and it becomes part of the replacement. `#define LOWER 0;`
turns `fahr = LOWER` into `fahr = 0;` which might work in some
places and silently break others. The preprocessor does not care.
It does not know what a statement is.

### Why ALL_CAPS?

Convention, not law. But a necessary one. Since `#define` has no
type safety and no scope, you need a visual flag that screams "this
is not a variable." ALL_CAPS is that flag. The tradition goes back
to the earliest Unix source. It stuck because it works.

### What about Rust in real time operating systems?

Most RTOS kernels are still C. FreeRTOS, Zephyr, ThreadX. Decades
of safety certifications built around C toolchains. Recertifying in
another language costs millions. But Rust is creeping in. New
projects like Hubris and RTIC. Bindings on top of C kernels. The
Embassy framework for embedded Rust. The argument is strong because
embedded systems often have no ASLR, no stack canaries, no MMU. A
buffer overflow there is not a crashed app. It is a safety incident.

## The Feynman Test

The preprocessor is a copy machine. Before your C code reaches the
compiler, the preprocessor walks through it and replaces every
`#define` name with its value. Literally. Text in, text out. It
does not understand C. It does not check types. It does not respect
scope. It just substitutes.

This is why it matters for security. The preprocessor operates on
trust. It trusts that the person who wrote the macro got it right.
It trusts that the substitution will not break something downstream.
It trusts that nobody will redefine the macro somewhere else. That
trust is the same kind of trust that let Heartbleed live in OpenSSL
for two years. The same trust that let Log4Shell sit in a logging
library for a decade. The same trust that almost let a backdoor
slip into every Linux distribution through XZ Utils.

Every layer of the machine was built by people who did not know what
they did not know. They made decisions that were reasonable in their
time. Those decisions became attack surfaces decades later. The
preprocessor does blind text replacement because in the 1970s that
was good enough. Integer division truncates silently because nobody
thought to warn you. Format strings accept user input because nobody
imagined someone would weaponize `printf`.

Reading K&R is not about learning to write C. It is about
understanding why vulnerabilities exist in the first place.

## What Is Next

Section 1.5. Character Input and Output. This is where programs
start reading from the user. `getchar()` and `putchar()`. File
copying, character counting, line counting, word counting. Multiple
subsections. It will probably take more than one day.

The machine stops being a calculator and starts being a listener.

---

*Day 4 of 365. The people who built the foundation could not see what
we would find in it.*
