---
title: "Day 28: The Gravity of the Stack"
date: 2026-04-14T06:00:00-05:00
draft: false
tags: ["c", "stack-frames", "buffer-overflow", "control-flow", "chapter-3", "hacker-track"]
description: "Chapter 3 closes. The stack frame opens. Logic lives in a physical place, and that place has gravity."
---

# Day 28: The Gravity of the Stack

Day 28 was a consolidation day. No new K&R material. The assignment was to sit with Chapter 3, close it properly, and then reason through a question I was left with overnight: what is a stack frame, mechanically, and why does the phrase "return address" appear in every buffer overflow write-up ever written?

I worked it out before opening a single reference. That was the point.

## What I Did

Chapter 3 review first. Three questions without the book:

Switch fall-through: execution slides into the next case unless a `break` forces an exit. The label is just an address. The CPU does not read intent.

`break` vs `continue`: `break` exits the loop. `continue` skips the current iteration and triggers the next condition test (or increment, in a `for`). One shreds the folder. The other just skips a page.

The `goto` concession: K&R discourages it broadly, but they admit it is the cleanest tool for escaping multiple nested loops at once. One jump outward versus a chain of flags and breaks.

All three solid. Chapter 3 is closed.

Then I turned to the overnight question.

## The Questions That Came Up

### What is a stack frame, and why does the return address matter?

I built the answer with a physical analogy before touching a diagram.

Imagine cooking dinner. That is `main`. You realize you need a spice and call a friend to go to the store. That is a function call. But you cannot just disappear. You have to freeze your current state so you can resume when the call returns.

You take a Manila folder and put four things in it:

- The shopping list (arguments passed to the function)
- Your kitchen timer and notes (local variables)
- A "where was I?" note (the return address, the exact line in the recipe you were reading when the phone rang)
- A pointer to the previous folder (the saved frame pointer, so the stack stays organized)

The stack is a literal pile of these folders on a desk. Push a new one when a function is called. Pop it when the function returns. You can only work with the folder on top. Everything else is buried.

When a function finishes, the program grabs the top folder, reads the return address, moves the instruction pointer there, and shreds the folder. The folder underneath becomes the active one. You are back in your kitchen on the correct line of the recipe.

That is the mechanism.

### Why does overflow reach the return address specifically?

This is where it gets precise.

Two facts in collision:

The stack grows downward. When a new frame is pushed, the stack pointer moves toward lower memory addresses. The frame for `main` is at a higher address than the frame for any function it calls.

Buffers fill upward. When you write into `char buf[8]`, index 0 is at the lowest address and index 7 is at the highest. Writing proceeds toward higher addresses.

The return address was placed at a higher address than your local variables when the frame was set up, before your buffer existed. It is sitting upstream.

The memory layout of a single frame looks like this:

```
[HIGHER ADDRESS]
  0x100C  Return Address        <- The "where was I?" note
  0x1008  Saved Frame Pointer   <- Link to previous frame
  0x1007  buf[7]                <- End of local array
  0x1006  buf[6]
  ...
  0x1000  buf[0]                <- Start of local array
[LOWER ADDRESS]
```

Write within `buf[0]` through `buf[7]` and you stay in your lane. Write a twelfth byte and you do not crash into the next function call (which is at a lower address). You write upward into 0x1008, overwriting the frame pointer, and then into 0x100C, the return address.

This is not an accident. It is topographical inevitability. The local variables are below the return address. The filling direction is upward. They are always on a collision course.

## The Feynman Test

A function call is a chore with a sticky note attached: "when you are done, come back here." The sticky note is the return address. The CPU reads it blindly when the function finishes and jumps to whatever address is written on it.

A buffer overflow is what happens when you write more data into a fixed-size container than the container can hold. The excess does not disappear. It lands in whatever memory sits next to the container.

Because of how the stack is laid out, "whatever sits next to the buffer" eventually includes the sticky note.

When an attacker overflows a buffer far enough, they overwrite the sticky note. Instead of "return to line 42 of main," the note now says "go run this other thing." The CPU does not know it has been rewritten. It follows the note. That is the exploit.

The return address is the holy grail because it controls the next instruction. Whoever controls the return address controls the program.

## Hacker Connection

Entry 32 added to the vulnerability notebook today: Implicit Fall-Through / Logic Bypass (CWE-484 / CWE-670).

`switch` labels are jump targets, not isolated blocks. Without an explicit `break`, execution continues linearly into the next case regardless of the label. The hardware sees one continuous instruction track. The programmer sees two separate branches. That gap is where the bug lives.

The concrete form is what I named the "Privilege Slide":

```c
switch (user_role) {
    case ADMIN:
        is_admin = 1;
        /* break omitted */
    case USER:
        access_level = READ_ONLY;
        break;
    default:
        access_level = NONE;
}
```

On Minion1, `case ADMIN:` is an address. If no break is present, the instruction pointer increments into `access_level = READ_ONLY`. The admin flag is set but the access level is not. The programmer intended two separate paths. The hardware executed one.

The `goto` variant is sharper: a jump that lands past an authentication check. The code reaches `run_privileged_command()` with the check never evaluated. Not because the check failed. Because it never ran.

The pattern is the same in both cases: the programmer trusts source code structure to enforce a boundary. The CPU does not read structure. It reads addresses. If there is no instruction forcing a boundary (a `break`, a `return`, a guard), the boundary does not exist at runtime.

## What Is Next

Chapter 4 opens tomorrow.

K&R is going to introduce functions, scope, external variables, and `static`. The question I am sitting with tonight: what does the compiler need to know about a function before it can call it? And what happens if it does not have that information yet?

That question leads somewhere. I will find out where tomorrow.

---

*Day 28 of 365. Data is just instructions waiting for a bad boundary. See you in Chapter 4.*
