---
title: "Day 007 Earning It Clean"
date: 2026-02-24T12:00:00-05:00
draft: false
tags: ["c", "knr", "state-tracking", "testing"]
description: "Rebuilding exercise 1-10 from scratch, word counting with state machines, and threat modeling a 20-line program."
---

# Day 007: Earning It Clean

Yesterday I copied code from Stack Overflow. Today I rebuilt it from nothing. The difference between those two days is the difference between knowing where something is and knowing what it does.

## What I Did

Started by rewriting exercise 1-10 from scratch. Replace tabs with `\t`, backslashes with `\\`, backspaces with `\b`. Make the invisible visible. First version used a flag variable. Declared `int esc = 0` inside the while loop, set it to 1 when a special character matched, checked it at the end to decide whether to print the original character. It worked. It was mine.

Then I looked at it and realized three separate `if` statements all get checked every time, even after a match. A character can only be one thing. So I rewrote it with `else if`. The flag disappeared. The final `else` handles normal characters. No wasted checks. Shorter and cleaner.

Read K&R 1.5.4 on word counting. The program tracks whether it is inside a word or outside a word using a state variable and two symbolic constants, IN and OUT. I recognized the pattern immediately. Same skeleton I had just built. Different purpose. The word counter increments `nw` on the OUT to IN transition. My word-per-line program from exercise 1-12 prints a newline on the IN to OUT transition.

Then I threat modeled the word counter. Twenty lines of code. How would I break it.

## The Questions That Came Up

### What is a magic number?

Any raw literal in code that hides its meaning. `if (state == 1)` forces the reader to guess. `if (state == IN)` reads like English. The compiler sees the same value either way because `#define` is text replacement. Magic numbers are for the machine. Symbolic constants are for the humans who come after you.

### What breaks the word counter?

The program checks for three whitespace characters. Space, tab, newline. But `\r`, `\v`, and `\f` are also whitespace. The program treats them as word characters. That is a real bug. A carriage return between two words and the counter thinks it is one word.

Punctuation creates a different kind of wrong. `mother-in-law` counts as one word. `hello...world` counts as one word. The program defines a word as anything that is not one of three specific characters. Useful definition. Not the human definition.

The counters are `int`. Feed it more than 2.1 billion characters and `nc` wraps negative. Integer overflow. Not buffer overflow. No string is stored, no buffer exists, but the counter itself has a ceiling nobody checks.

## The Feynman Test

State tracking is a variable that remembers where you are. Inside a word or outside a word. That is all. When you see a letter after whitespace, you just crossed a boundary. When you see whitespace after a letter, you crossed back. The variable holds which side of the line you are standing on.

The power is in what it prevents. Five spaces in a row after a word. The first space flips the state to OUT and prints a newline. The next four spaces see OUT and do nothing. One transition, one action, no matter how many spaces pile up. Without the state variable you would print five newlines and the output would be wrong.

Every protocol parser, every packet inspector, every input validator I have worked with in 21 years uses this same idea. State machines are not a theory exercise. They are how real systems decide what to do next.

## What Is Next

The rest of the exercises in 1.5 and then into 1.6 on arrays. Arrays are where C starts getting dangerous. Memory laid out in a row with nothing to stop you from walking past the end. Every buffer overflow I have ever investigated started there.

---

*Day 7 of 365. Yesterday I borrowed an answer. Today I built three.*
