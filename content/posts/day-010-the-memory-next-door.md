---
title: "Day 010: The Memory Next Door"
date: 2026-02-27T12:00:00-05:00
draft: false
tags: ["c", "arrays", "gdb", "off-by-one", "functions", "security"]
description: "K&R 1.6 and 1.7. Arrays, character arithmetic, functions. And the first time I watched a bug disappear into memory that wasn't mine."
---

# Day 010: The Memory Next Door

K&R 1.6 introduces arrays. Ten integers sitting next to each other
in memory. No gaps. No names. Just addresses and math.

It also introduced my first real bug hunt in GDB.

## What I Did

Three exercises today. The digit counting program from 1.6 led to
exercise 1-13, a histogram of word lengths. Then 1-14, a histogram
of character frequencies. Then 1.7 on functions, which gave me
exercise 1-15, refactoring the Fahrenheit to Celsius table with a
conversion function.

The digit counter taught me the `c - '0'` pattern. A character
minus another character gives you an integer. `'5' - '0'` is 5.
It works because the digits are contiguous in ASCII. I already
knew the anchor points from Day 6. Today I used them.

For 1-14 I adapted the same trick. `c - '!'` maps every printable
ASCII character to a zero-based index. `!` is 33. `~` is 126.
That gives 94 characters. I wrote the size as `('~' - '!' + 1)`
so the compiler does the math and anyone reading the code can see
the range.

For 1-15 I wrote a `toCelsius` function and validated it with
negative 40. That is where Fahrenheit and Celsius cross. Both
columns read negative 40. The math works.

## The Questions That Came Up

### Why no exponentiation operator?

C gives you what the CPU gives you. Add, subtract, multiply,
divide. Those are hardware instructions. Exponentiation is not.
It lives in `math.h` as a library function. C was built to stay
close to the machine. That is not a limitation. That is the point.

### What happens when source files go missing?

The compiler processes each file. The linker stitches them together.
A missing file means undefined references at link time. Build
systems like `make` track these dependencies. That comes later
in K&R.

### Old-style function declarations?

K&R mentions a "transition period" for the old declaration syntax.
That period lasted about 30 years. C99 killed implicit int. C23
removed K&R-style definitions entirely. My compiler will not
accept the old form. I already learned this on Day 2 when it
rejected code without `int main`.

## The Feynman Test

An off-by-one error is writing to the slot next to the last one
you were allowed to use. Imagine a bookshelf with five slots
numbered zero through four. You write a loop that fills slots
zero through five. Slot five does not belong to you. It belongs
to whatever is next to the bookshelf.

In a computer, "whatever is next" might be another variable. It
might be a return address that tells the program where to go when
a function finishes. The machine does not check. It writes where
you point it. If you point one slot past the end, it writes there
silently.

The dangerous part is that it often works. The compiler might put
padding between your shelf and the next one. Tests pass. The
program runs for years. Then someone recompiles with different
settings and a variable lands where the padding used to be. The
bug that was invisible becomes a crash. Or worse, an attacker
figures out what sits past the boundary and writes something
there on purpose.

I saw this today. Not in a textbook. In GDB.

## Hacker Connection

I wrote `i <= SIZE` instead of `i < SIZE` in my array
initialization loop. An array of 5 integers, indexed 0 through 4,
and my loop wrote to index 5.

I loaded it into GDB and printed the addresses of every local
variable. Built a map of the stack frame:

```
df80  lengths[0]
df84  lengths[1]
df88  lengths[2]
df8c  lengths[3]
df90  lengths[4]
df94  lengths[5]  <- out of bounds
df98  ???
df9c  c
dfa0  state
dfa4  count
dfa8  j
dfac  i
```

The write landed at `df94`. In this compilation the compiler had
placed padding there. Nothing visible broke. No crash. No wrong
output. The bug was silent.

That is the pattern behind CVE after CVE. Off-by-one errors in
array bounds. They survive testing because the memory layout
happens to absorb the damage. Then something changes and they
do not.

I also fixed a state machine bug in 1-13 where consecutive spaces
created ghost words. The fix was checking for a transition from IN
to OUT rather than just checking if the state was OUT. Same pattern
from exercise 1-12. Same mistake. Now it is a pattern I recognize.

## What Is Next

K&R 1.8 on character arrays. Then 1.9 on external variables and
scope. That closes out Chapter 1. The hacker track got real work
today with the GDB session. The vulnerability pattern notebook has
a new entry: off-by-one on array bounds, `<=` vs `<`.

---

*Day 10 of 365. The most dangerous bugs are the ones the tests
never catch.*
