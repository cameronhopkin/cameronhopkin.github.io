---
title: "Day 026: The Negative Domain"
date: 2026-04-09
tags: ["c", "security", "operation-plus-ultra", "k&r"]
---

I came back after 17 days off expecting to be rusty. The Feynman test said otherwise. Turns out the concepts stuck. What didn't stick was the habit of showing up. That's a different problem.

---

## What I Did

Closed out the `goto` overnight question, worked through sections 3.5 and 3.6, wrote `reverse`, `expand`, and the `itoa` INT_MIN fix. Four exercises committed to Minion1. Two new notebook entries.

The `expand` function had a buffer overflow baked in that `-Wall -Wextra -Werror` couldn't see. The compiler checks syntax. It doesn't know that `!-~` is 3 characters going in and 94 coming out. That's the Range Bomb. Entry 29.

The `itoa` fix was the session's real work. K&R's version breaks on `INT_MIN`. The standard fix is to cast to a wider type. The K&R-idiomatic fix is to never leave the negative domain at all.

---

## Questions That Came Up

I assumed `longjmp` and `throw` were essentially the same mechanism at different abstraction levels. They're not. `longjmp` restores CPU state and discards everything in between. No destructor calls. No resource cleanup. Just a teleport that leaves the luggage on the platform. `throw` stops at every station on the way up. That distinction is the difference between a debugging tool and a memory leak factory.

I also caught myself referencing Entry 28 before the code existed on Minion1. That's a forward reference. At Day 365 the notebook has to be an audit trail of work performed, not work planned. I stopped, read 3.6, then wrote the code.

---

## The Feynman Test

Most people think integers are a balanced seesaw. For every positive number, a negative mirror. That's not how two's complement works. The negative side has one extra value. `INT_MIN` exists. Its reflection doesn't.

When I first learned this, the instinct was to escape the problem. Cast to `long`. Use `unsigned`. Get out of the danger zone. But the idiomatic C solution goes the other direction. Stay negative. Work in the domain that actually fits. `n % 10` on a negative number gives a negative remainder. `'0' - (-8)` gives `'8'`. The machine's asymmetry becomes the safe harbor instead of the trap.

That's the lesson from today. Sometimes the right move isn't to avoid the dangerous terrain. It's to understand it well enough to live in it.

---

## Hacker Connection

The Range Bomb in `expand` is CWE-120. A 3-character input exploding into 94 characters of output is exactly how compressed or encoded payloads slip past length-based input filters. The filter checks the input. The overflow happens on the output. Static analysis misses it. The auditor misses it if they're only looking at the function signature. The check has to be inside the loop, per write, not at the door.

---

## What Is Next

Section 3.7. Break and continue. I know the surface behavior. The overnight question is about the precise difference in a nested loop context and which one K&R is uncomfortable with.

Every day the Danger Map gets more detailed.
