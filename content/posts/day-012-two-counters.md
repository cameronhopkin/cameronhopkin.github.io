---
title: "Day 012 Two Counters"
date: 2026-03-01T08:00:00-05:00
draft: false
tags: ["c", "knr", "character-arrays", "gdb", "off-by-one"]
description: "Exercises 1-16 through 1-18. Dual counters, GDB investigations, and what happens when zero means two things."
---

# Day 012: Two Counters

Three exercises today. All built on the `getline` and `copy` pattern
from yesterday. All forced me to think about what happens when reality
exceeds the space you gave it.

## What I Did

Exercise 1-16 asks you to handle arbitrarily long input lines. The
original K&R longest-line program silently truncates and lies about
the length. The fix needs two counters. `i` tracks the actual length
of the line. Every character increments it. `j` tracks the buffer
position. It stops when the buffer is full. The function returns `i`
so the caller gets truth. The buffer only holds what fits safely.

That separation between "how big is this really" and "how much can I
safely store" is the question every input handler has to answer.
When those two values diverge, you have a problem. Or an
opportunity, depending on which side of the keyboard you sit on.

I also hit a name collision. `getline` is a POSIX standard library
function now. The compiler rejected it. Renamed to `get_line` and
moved on. Function name collisions are a real bug class. Imagine
accidentally shadowing a security-critical library function and
nobody notices until production.

Exercise 1-17 prints lines longer than 80 characters. Simple filter.
`get_line` already returns true length. Define `THRESHOLD` as 80 and
check against it. But then I tested with exactly 80 characters and
it printed. Off by one. `get_line` counts the newline, so 80 visible
characters plus `\n` equals 81, and `81 > 80` is true. Fixed it to
`len - 1 > THRESHOLD` so the comparison works on visible characters.

Then I wanted to see it for myself.

Loaded the program into GDB. Created a test file with exactly 81 A's
and no newline. Set a breakpoint on `get_line`, ran it, used `finish`
to let the function complete, and checked `$eax`. It said `0x51`. That
is 81 in hex. `len - 1` gives 80. `80 > 80` is false. The program
correctly produces no output.

I could have trusted the math. I asked the machine instead.

Exercise 1-18 removes trailing blanks and tabs and deletes blank
lines. Walking the string backward with a `for` loop felt natural.
Start at the end, check for spaces and tabs, replace with `'\0'`
until you hit a real character. Efficient. No wasted passes.

Then I tested with blank lines in the middle of the input and
everything after them vanished. The bug was subtle. My first version
of `get_line` did not store the newline. A blank line returned 0.
The `while` loop in `main` treated 0 as EOF and quit early. The
program never saw the lines after the blank ones.

Zero meant two things. Empty line and end of file. Same return
value, different realities. The program could not tell them apart.

Fixed it by putting the newline back in `get_line`. Now a blank
line returns 1. The loop survives. `trim` strips the newline along
with other trailing whitespace. Returns 0 after trimming. The
`if (len > 0)` guard suppresses the blank line. EOF still returns 0
from `get_line` because there is no newline to store. The two cases
are distinguishable again.

## The Questions That Came Up

### Why does the newline-terminated edge case matter?

Because every protocol, file format, and input stream has to decide
how it signals "end of record" versus "end of stream." When those
two signals can be confused, parsers break. I have seen this in log
ingestion pipelines, HTTP chunked encoding, and certificate parsing.
The details change. The pattern does not.

### Why walk backward for trimming?

Walking forward means scanning the entire string to find the end,
then backtracking. Walking backward from the known length gets to
the trailing whitespace immediately. In C, where you are responsible
for every operation, the efficient approach is the correct approach.

## The Feynman Test

Imagine you are filling out a form and the "Name" field allows 20
characters. Your name is 25 characters long. A good form tells you
"your name is 25 characters but we can only show 20." A bad form
silently chops your name at 20 and pretends that is all you typed.
The dual counter is the difference between those two forms. One
counter tracks reality. The other tracks capacity. When they
disagree, the honest program tells you both numbers.

## Hacker Connection

Sentinel value collisions. When a function uses the same return
value to mean two different things, the caller cannot distinguish
between them. In my exercise, 0 meant both "empty line" and "no
more input." In real systems, this pattern shows up everywhere.
`malloc` returns `NULL` for both "zero bytes requested" and
"allocation failed." Old APIs return -1 for errors but also for
legitimate values. `strstr` returns `NULL` for "not found" which
is the same pointer value as a null pointer dereference target.

Every ambiguous sentinel is a potential logic bug. Every logic bug
in input handling is a potential vulnerability. The fix is always
the same: make sure every distinct condition has a distinct signal.

## What Is Next

Section 1.10 on scope and external variables. This is the last
section before Chapter 1 ends. Then we close out Phase 1's K&R
track and Chapter 2 begins. The vulnerability pattern notebook gets
a new entry today: sentinel value collisions.

---

*Day 12 of 365. When zero means two things, the program believes
whichever one it hits first.*
