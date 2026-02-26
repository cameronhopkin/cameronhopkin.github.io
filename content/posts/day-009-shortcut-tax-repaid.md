---
title: "Day 009: Shortcut Tax Repaid"
date: 2026-02-26T18:00:00-05:00
draft: false
tags: ["c", "knr", "exercises", "accountability"]
description: "Rebuilding ten exercises from scratch on a different machine. No peeking. No shortcuts. Proof of ownership."
---

# Day 009: Shortcut Tax Repaid

On Day 6 I copied exercise solutions from Stack Overflow. Got
caught. Wrote about it. Said I would do better. Today was the
receipt.

## What I Did

Ten exercises. 1-3 through 1-12. All rewritten from scratch on
a Linux machine. No peeking at the versions on my normal workstation.
No Stack Overflow.
Just the exercise prompts from K&R and whatever I actually
understood.

Some of them came fast. The temperature table exercises were muscle
memory at this point. Print a heading. Reverse the loop. Change the
formula. Those took seconds.

The verification exercises made me think. Printing the value of
`getchar() != EOF` required understanding that a comparison in C
is an expression that evaluates to 0 or 1. Printing EOF itself
gave me `-1` and reminded me why we declare the input variable as
`int` instead of `char`. A `char` cannot hold -1 without collision.

Exercise 1-9 was the flag variable pattern. Track whether the last
character was a blank. Only print a blank if the previous character
was not one. The sequencing matters. Reset the flag, check the flag,
set the flag. That order is not arbitrary.

Exercise 1-10 was the redemption round. This is the one I copied on
Day 6. Today I wrote it clean. Tabs become `\t`, backspaces become
`\b`, backslashes become `\\`. Testing it taught me something new.
Typing a tab in the terminal triggers shell completion instead of
sending a literal tab character. Had to use `echo -e` with escape
sequences piped into the program to actually verify it worked.

Exercise 1-12 took thirty minutes. The state machine. IN and OUT
with `#define`. Read a character. If it is not whitespace and the
state is OUT, transition to IN. If it is whitespace and the state
is IN, print a newline and transition to OUT. I knew the pattern
but rebuilding it without looking at my old code forced me to think
through every condition again. That was the point.

After the exercises I set up proper repo hygiene. Created a `build/`
directory for compiled binaries. Added a `.gitignore` so executables
never touch version control. Cleaned up inconsistent file naming.
Pushed everything to GitHub.

## The Questions That Came Up

Testing programs that read from stdin is harder than I expected. You
cannot just type a tab and expect the terminal to pass it through.
The shell intercepts it. Piping input with `echo -e` or `printf`
solves this but it was not obvious at first.

The `else` versus `else if` bug bit me on exercise 1-8. Writing
`else (c == '\n');` instead of `else if (c == '\n')` compiled
without errors. The compiler saw a valid expression statement. But
the logic was completely wrong. The `++nl` counter ran on every
character instead of only newlines. Syntactically legal. Logically
broken. Silently wrong.

## The Feynman Test

A state machine is a program that remembers one thing: what mode it
is in. The word-per-line program only needs to know whether it is
currently inside a word or outside one. Every character it reads
either keeps it in the same mode or flips it to the other one.

Think of a light switch. You walk through a room and every time you
cross a doorway you flip the switch. You do not need to remember
every doorway you passed through. You just need to know whether the
light is on or off right now. That one piece of information controls
your next action.

The power of this pattern is that it turns complicated input into
simple decisions. No matter how many spaces or tabs or newlines come
in a row, the program only prints one newline because it only reacts
to the transition from IN to OUT. Not to being OUT.

## Hacker Connection

The `else` bug in exercise 1-8 is the same class of error as
Apple's "goto fail" vulnerability from 2014. Code that compiles
and runs but does something the programmer did not intend. In that
case a duplicated `goto fail;` line caused SSL certificate
validation to always succeed. The compiler did not complain. The
tests did not catch it. The code shipped.

The state machine pattern shows up everywhere in security tooling.
Firewalls use state tables to track TCP connections. IDS engines
track protocol states to detect anomalies. Parsers use state
machines to process input safely. Getting the transitions wrong
in any of those contexts means either dropping legitimate traffic
or letting malicious traffic through. The pattern is simple. Getting
it right under every edge case is where the discipline lives.

## What Is Next

Section 1.6 on arrays. This is where C introduces indexed data in
contiguous memory. Digit counting with `ndigit[10]`. The expression
`c - '0'` to convert a character to its numeric value. This is the
foundation for understanding buffer overflows and off-by-one errors.

The hacker track stays warm. Lab is operational. Exercises are clean
and pushed. Tomorrow the new material starts again.

---

*Day 9 of 365. The shortcut tax was three days of doubt. The
repayment took two hours of honest work.*
