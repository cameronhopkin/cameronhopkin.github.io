---
title: "Day 024: The Illusion of Structure"
date: 2026-03-20T21:46:00-05:00
draft: false
tags: ["c", "control-flow", "switch", "dangling-else", "knr", "chapter-3"]
description: "Chapter 3 opens. The code looks right. The compiler disagrees."
---

# Day 024: The Illusion of Structure

Chapter 3 started tonight. I expected control flow to feel familiar.
It does. That is the problem. Familiarity is where you stop reading
carefully. Tonight was a reminder that the way code looks on screen is
often a story the programmer told themselves. The compiler is reading
something else entirely.

## What I Did

Started by clearing the overnight question from Day 23: switch
fall-through. Then Section 3.1 and the dangling else. Then a derivation
on why `continue` breaks the for/while equivalence. Three concepts. All
of them about the gap between what you see and what the machine does.

The switch work came first. Pulled it apart down to the assembly level.
Each `case` is a label. An address. When the match fires, execution drops
onto that address and keeps moving. There is no door to close behind you.
Fall-through is not a feature the compiler adds. It is what happens when
the compiler adds nothing. The `break` is the injected jump. Fall-through
is just gravity.

Then the dangling else. Typed the nested `if` examples from 3.1. Watched
the indentation lie to me in real time. The rule is mechanical: an `else`
attaches to the closest previous `if` in the same block that does not
already have one. Whitespace is invisible to the parser. The code can
look like it handles a failure case while the compiler has attached that
handler somewhere else entirely.

The for/while/continue derivation was prior knowledge, not a read-ahead.
I knew `continue` skips the rest of a loop body. I worked out why it
breaks the equivalence from the structure of the two loops. In a `for`
loop, the increment step is part of the loop control, not the body.
`continue` is routed through it on the way back to the condition check.
In a `while` loop, the increment lives at the bottom of the block.
`continue` jumps over it. Same keyword. Different machine behavior.

Notebook Entry 22 went in: the Dangling Else as a structural
vulnerability. Initial CWE was wrong (698). Corrected to CWE-670,
Always-Incorrect Control Flow Implementation. The root cause is a logic
error that silently skips a protection branch. That deserves the right
number.

## The Questions That Came Up

### Is fall-through visible in the disassembly?

No. That is the point. You will not see fall-through in the assembly. You
will see the absence of a jump. The `break` compiles to an unconditional
jump past the switch body. Fall-through is what the code looks like when
that instruction is missing.

### What does "closest if" mean at the token level?

It means the parser does not look at indentation, comments, or intent.
It scans backward through tokens and attaches the `else` to the first
unmatched `if` it finds in the current block. The human reads the layout.
The parser reads the grammar.

## The Feynman Test

A switch statement is a ladder. Each `case` is a rung with a label
painted on it. When your value matches a label, the program drops you
onto that rung. Then it keeps falling. There are no floors between the
rungs. No walls between the cases. The only thing that stops the fall is
a `break`, which is just an instruction that says jump to the end.
Without it, the program executes everything below the match in order
until it runs out of ladder.

This is not a bug. It is a design. C was built to be close to the
machine. In assembly, labels are addresses. Fall-through is the default
because stopping requires work. C does not do work you did not ask for.

The security implication is about auditing. You cannot skim a switch
statement. You have to verify every case has a `break`, or verify that
the fall-through is intentional and documented. One missing `break` in
an authentication handler is a logic error that hands execution to the
wrong branch. The code will look fine. It will compile clean. It will
run. It will just do something other than what the developer intended.

## Hacker Connection

The dangling else is a Default Permit vulnerability. A developer writes
an outer check for authentication and an inner check for privilege level.
They indent an `else` to handle the unauthenticated case. The compiler
attaches it to the inner check. When an unauthenticated user hits the
outer condition, the failure branch never fires. No redirect. No log.
Execution stops and whatever privileged state existed before the check
remains in place.

This is CWE-670 in practice. The control flow is structurally wrong in a
specific, reproducible way. The fix is braces. Not indentation. Braces
change the grammar and force the else to attach to the correct parent.
One character pair that closes an entire class of logic errors.

Entry 22 is in the notebook. No nested `if` ships without explicit
scoping.

## What Is Next

Sections 3.3 and 3.4. Else-if chains and switch in full. K&R's treatment
of fall-through comes after I derived the mechanics myself. I want to see
what I missed.

Hacker track: sitting with whether the continue/equivalence break earns
its own entry. A skipped increment in a security-critical loop has a
shape. I want to find it before I write it down.

---

*Day 24 of 365. The compiler does not read your indentation. Neither
should you.*
