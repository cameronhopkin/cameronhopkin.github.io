---
title: "Day 003 The For Statement"
date: 2026-02-20T18:00:00-05:00
draft: false
tags: []
description: ""
---

# Day 003: The For Statement

Today I opened K&R section 1.3 and met the for statement. Then I went
back and knocked out three exercises from 1.2 that I had skipped. Three
variations on the same temperature conversion program. Each one broke
something different.

## What I Did

Exercise 1-3 asked for a heading above the temperature table. I
overthought it. Tried using format specifiers and variable references
when all it needed was plain text in a printf. Not every problem needs
a clever solution.

Exercise 1-4 was reversing the formula. Celsius to Fahrenheit instead
of the other way around. The algebra came easy.
`fahr = (celsius * 1.8) + 32`. Getting it into C without errors was
the fight. I kept writing `x` for multiplication instead of `*`. Muscle
memory from math class versus what C actually wants.

I also learned that a semicolon after `int main()` turns a function
definition into a declaration. The compiler sees `int main();` and
thinks you are announcing a function that exists somewhere else. Then
it hits the opening brace and has no idea what to do with it. One
character. Completely different meaning.

Exercise 1-5 was printing the table in reverse order. 300 down to zero.
I figured that one out on my own. Changed the starting value, flipped
the condition from `<=` to `>=`, subtracted instead of added. Same
structure, different direction. That felt good.

## The Questions That Came Up

### Do while and for compile to the same thing?

The for statement puts initialization, condition, and increment on one
line. The while loop scatters them across three. The compiler does not
care. It produces the same machine code either way. The for loop is for
the human reading it later.

### Do spaces take up memory?

If spaces and formatting do not change how the program runs, do they
cost anything? They do not. The compiler strips all whitespace before
it builds the binary. Spaces, tabs, blank lines exist for humans. The
machine never sees them.

### Vimtutor observations

I continued vimtutor from yesterday. Capital letters do entirely
different things than lowercase. `a` and `A` are two separate commands.
Hand placement feels different when you commit to never touching the
mouse. H for moving left is going to take time. There are two modes so
far: insert and normal. `dw` only works if you are at the start of a
word. My nvim setup helps by showing which mode I am in. Stopped at
lesson 1.3.1, the put command.

## The Feynman Test

What is the difference between a while loop and a for loop in C?

There is no difference to the machine. A for loop is a while loop with
the bookkeeping collected in one place.
`for (fahr = 0; fahr <= 300; fahr = fahr + 20)` does exactly what the
while version does: sets a starting value, checks a condition before
each pass, and increments at the end of each pass. The for version
makes the whole structure visible in a single line so the next person
reading your code can see the logic without hunting for it.

Why does this matter for security? Because readable code is auditable
code. When you are reviewing someone else's C for vulnerabilities, a
for loop tells you the bounds immediately. A while loop makes you
search for the initialization and the increment separately. Same
logic, different cognitive cost. The easier code is to read, the
harder it is for bugs to hide.

## What Is Next

Section 1.4 on symbolic constants. I do not yet know what problem they
solve that variables do not. That question is sitting in the notebook
waiting.

---

*Day 3 of 365. The compiler does not care about style. The humans who
read your code do.*
