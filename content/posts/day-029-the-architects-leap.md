---
title: "Day 29: The Architect's Leap"
date: 2026-04-15T00:00:00-05:00
draft: false
tags: ["c", "functions", "ABI", "type-confusion", "K&R", "chapter-4"]
description: "K&R 4.1 — what happens when the compiler builds a bridge without a blueprint."
---

# Day 29: The Architect's Leap

Chapter 4 opens with functions. K&R calls them the building blocks of C programs — the mechanism by which large problems get broken into smaller, composable pieces. That framing is clean and correct. But the section I kept staring at wasn't the function definition syntax. It was the implied question underneath it: what happens when the caller and the callee don't share a contract?

## What I Did

Read K&R 4.1. Typed the `strindex`/`getline`/`main` pattern-search program by hand. Compiled it, ran it, traced the logic. Then built a minimal reproducer to test the implicit declaration behavior the section describes — a function called before it is declared, with a mismatched return type.

Compiled with `-Wall -Wextra -Werror`. The terminal did not stay quiet.

```
ex4-01.c: In function 'main':
ex4-01.c:5:18: error: implicit declaration of function 'add_numbers'
ex4-01.c:10:8: error: conflicting types for 'add_numbers'; have 'double(double, int)'
ex4-01.c:5:18: note: previous implicit declaration of 'add_numbers' with type 'int()'
```

The compiler told me exactly what it assumed, exactly where it assumed it, and exactly where the assumption broke. `int()` — the default signature. The ghost contract. The bridge built from the wrong side of the canyon.

Added the forward declaration. Recompiled. Clean.

## The Questions That Came Up

**What does the compiler actually assume when it sees a call with no prototype?**

It assumes `int` return type. It promotes all arguments via default rules: `char` and `short` become `int`, `float` becomes `double`. It emits the `CALL` instruction and leaves the linker to find the symbol. It does not stop. It does not warn, in C89 mode.

**Where exactly does the mismatch live in the hardware?**

This is the piece worth holding. On x86_64, integer return values come back in `RAX`. Floating-point return values come back in `XMM0`. When the compiler implicitly declares a function as returning `int`, it emits code that reads the return value from `RAX`. When the actual function definition returns `double`, it writes to `XMM0`. The caller looks in `RAX` and finds whatever ghost data was left there from before. No crash. Just wrong.

The mismatch is bidirectional. Argument path and return path both break. The caller may not set up `XMM` registers at all if it thinks it's passing ints — the callee looks for its `double` in `XMM0` and finds garbage.

## The Feynman Test

You ordered a pizza. Large pepperoni. But the phone connection was bad, and what they heard was "small plain." You both hang up thinking you're in agreement. When the box arrives, the lid doesn't even fit the order.

That's implicit declaration. The caller and the callee each believe they know what was agreed. Neither one checked. The contract was never signed — it was assumed. When the order arrives at runtime, the kitchen is reading from a different shelf than the one the front desk labeled.

The fix is one line. A prototype. `double add_numbers(double, int);` placed above `main`. That line is the shared blueprint — the commitment that both sides agreed before any building began.

## Hacker Connection

This goes into the notebook as Entry 33: ABI Contract Violation.

The vulnerability class is type confusion at the calling convention level. In legacy codebases compiled without strict prototype enforcement, an implicit declaration creates a silent disagreement about where return values live in the register file. If the corrupted return value controls allocation size — `malloc(add_numbers(...))` — the heap gets a garbage size. If it controls a loop bound or a privilege check, the downstream behavior is determined by whatever happened to be in `RAX` when the function returned.

This is not theoretical. C89 mode and pre-ANSI codebases compiled this silently for decades. The fix that actually works is `gcc -Wimplicit-function-declaration -Werror` as a hard build requirement — not a style suggestion, a contract enforcement mechanism.

## What Is Next

4.2 covers functions returning non-integers. The section where K&R formalizes the prototype syntax and explains why the compiler needs it. After the implicit declaration reproducer today, that section is going to land differently than it would have cold.

Open question to sit with: K&R introduces `extern` declarations in 4.3 for variables shared across files. The `extern` keyword makes storage explicit. What does that visibility model mean for the hacker track — specifically, what attack surface does a shared external variable create that a local variable does not?

---

*Day 29 of 365. The compiler doesn't ask for a blueprint. You have to hand it one.*
