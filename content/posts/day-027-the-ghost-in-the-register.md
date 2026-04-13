---
title: "Day 27: The Ghost in the Register"
date: 2026-04-13T00:00:00-05:00
draft: false
tags: ["C", "systems", "control-flow", "security", "K&R"]
description: "K&R 3.7 and 3.8. break, continue, and the one place K&R says goto is okay."
---

# Day 27: The Ghost in the Register

K&R 3.7 and 3.8 today. Short sections. Dense ideas. The kind of material that feels obvious until you have to explain it out loud and realize you were holding a vague picture, not a real one.

## What I Did

Worked through sections 3.7 and 3.8. Break and continue first, then goto.

Before opening the book I had to answer the overnight question: what is the precise difference between break and continue in a nested loop, and which one does K&R have reservations about.

I reasoned through it before reading. The mechanics matched. The reservation is about continue, not break. K&R's point is that continue forces the reader to track a jump that the eye doesn't naturally follow. In a for loop, continue sends you backward on the page to the increment in the header. The code reads top to bottom. The machine doesn't.

Then 3.8. The goto question. K&R defends it in exactly one place: escaping deeply nested loops when the alternative is a flag-and-ripple across every level of nesting. Three loops deep, something goes wrong, break only gets you one level out. You either carry a flag through every parent loop or you jump. K&R picks the jump.

I missed one legitimate use case in my prediction. K&R also defends goto for consolidating cleanup. One label at the end of the function. Every early exit jumps there. Free the memory, close the file, leave once. Without goto you duplicate that cleanup at every exit point. That pattern is everywhere in the Linux kernel.

## The Questions That Came Up

### Does break in a switch inside a while loop stop the while?

No. It stops the switch. The while keeps running. The switch exits clean but the outer loop has no idea anything happened. If the programmer expected the loop to stop, it won't. It keeps going. Whatever state the switch left behind, the while inherits it.

### Is there a real-world case where this causes problems?

Yes, and I've seen it. Elasticsearch agent zombie state. An inner operation fails, break exits only the inner context, the outer scheduler doesn't know to stop. It keeps spawning agents. The agents have nothing to work with. The dashboard reports data. The data is stale. You have to go to the command line to see what's actually happening.

### Why is continue safer in a for loop than a while loop?

Because the increment lives in the for header and cannot be skipped. In a while loop, the increment is in the body. Continue skips the body. If the counter is below the continue statement, it never runs. Infinite loop. The for loop's structure makes that mistake structurally impossible.

## The Feynman Test

Imagine a building with rooms inside rooms. You're in the innermost room. Break opens the door and puts you in the hallway of the room you just left. Not outside the building. In the hallway. The outer room is still there. It doesn't know you had a problem. It just keeps doing its thing.

Continue is stranger. In a for loop, continue doesn't put you in the hallway. It teleports you to a specific spot in the outer room that you already passed. You jump backward on the floor plan to a step you thought you were done with. The floor plan reads left to right, top to bottom. The teleportation goes against that.

That's the readability problem K&R is pointing at. Not that continue is wrong. That it makes the reader track a jump that doesn't match how we read code.

## Hacker Connection

Two entries in the notebook today.

Entry 30 is Scope-Limited Control Binding. The security failure is the zombie loop. Inner operation fails, break exits the wrong scope, outer loop continues with partial or invalid state. This shows up in parsers, protocol handlers, anything with nested state machines. If the programmer expected a global exit and got a local one, the program keeps processing data it shouldn't touch.

Entry 31 is Non-Local Function Jump. The goto pattern. The security angle here is what goto lets you skip. In C99, jumping over a VLA declaration is illegal. In earlier C, jumping into the middle of a block that assumes initialized state is just undefined behavior with no compiler enforcement. The programmer thinks the variable was set up. It wasn't. The compiler didn't stop it.

The cleanup-goto pattern is the defensive side. One exit point, one place to free memory, one place to close file handles. If you duplicate cleanup at every early return and miss one, you have a resource leak. Or worse, a double free if a later refactor adds a second path to the same cleanup code.

## What Is Next

Day 28 is a review day. Chapter 3 consolidation before Chapter 4 opens Wednesday.

Open question going into the review: Chapter 4 is functions and scope. The hacker track implication is the stack frame. Every function call is a frame. The frame has a return address. I know where this is going. I want to make sure the K&R mechanics of function calls and scope are solid before the exploitation track starts building on them.

---

*Day 27 of 365. The machine follows the innermost rule. If you want further, you have to say so explicitly.*
