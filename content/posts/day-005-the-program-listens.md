---
title: "Day 005 The Program Listens"
date: 2026-02-22T21:00:00-05:00
draft: false
tags: ["c", "getchar", "precedence", "K&R"]
description: "C stops printing and starts listening. getchar, putchar, and the precedence trap that has ruined code for fifty years."
---

# Day 005: The Program Listens

Four days of telling the computer what to print. Today it learned to listen. K&R section 1.5 introduces character input and output. Two functions. getchar() reads one character. putchar() writes one character. Everything else is built from those two.

## What I Did

Started with the file copying program from 1.5.1. Six lines. Reads a character, checks if it is EOF, prints it, loops. The simplest useful program in the book so far. I compiled it, piped text through it, ran it interactively, and sent Ctrl+D to kill it.

Then the exercises. Exercise 1-7 asked for the value of EOF. One printf call. The answer is -1. That is why getchar() returns an int and not a char. A char cannot hold -1 and every valid character at the same time. The types matter.

Exercise 1-6 asked me to verify that `getchar() != EOF` evaluates to 0 or 1. Printed the expression directly. Fed it a character, got 1. Sent EOF, got 0. No booleans in C. Just integers pretending.

Moved into 1.5.2, character counting. Short section. The increment operator, the long type, the for loop with an empty body. That empty body tripped me up. A for loop with nothing but a semicolon where the work should be. C requires a statement to satisfy the grammar. A bare semicolon is a null statement. It counts.

Still writing pre-C99 code to match K&R. Every compiler warning is a lesson in what the standard eventually fixed and why.

## The Questions That Came Up

### Precedence

The big one. The `!=` operator binds tighter than `=`. Without parentheses, `c = getchar() != EOF` evaluates the comparison first and assigns c the value 0 or 1. The actual character disappears. K&R force the parentheses: `(c = getchar()) != EOF`. That is not style. That is survival.

I wanted the full precedence table. K&R says Chapter 2. So I wrote it in the notebook and moved on.

### Where other languages learned

C has `==` for equality and `=` for assignment. One character difference. Python blocks assignment inside conditions entirely. Rust makes the compiler reject it. Both languages exist partly because C let you shoot yourself in the foot and called it a feature.

### The empty body

A for loop needs a body. A semicolon alone satisfies that requirement. Useful for loops that do all their work in the condition or increment expression. Dangerous for the same reason. An accidental semicolon after a loop header and the body never runs. Same mechanism, opposite disaster.

## The Feynman Test

Precedence is the order the machine resolves ambiguity.

When you write `c = getchar() != EOF`, there are two operations. Assignment and comparison. The machine has to pick which one happens first. In C, comparison wins. So the machine compares the character to EOF, gets a 0 or a 1, and stores that in c. The character you actually read is gone.

The fix is parentheses. `(c = getchar()) != EOF` forces the assignment first. Now c holds the real character. Then the comparison checks it against EOF.

This matters for security because precedence bugs are silent. The code compiles. It runs. It even looks correct at a glance. But the data flowing through the program is not what you think it is. The input was a character. By the time it reaches your variable, it has been quietly reduced to a bit. Information vanished between two operators, and nobody saw it happen.

## What Is Next

Section 1.5.3. Line counting. The newline character as something you can test for. The equality operator in practice. Exercise 1-8 asks me to count blanks, tabs, and newlines. First program that has to distinguish between types of input. First real branching decision.

---

*Day 5 of 365. The program listened. Then precedence made sure I was listening too.*
