---
title: "Day 006 The Shortcut Tax"
date: 2026-02-23T12:00:00-05:00
draft: false
tags: ["C", "K&R", "exercises", "accountability"]
description: "Line counting, state tracking, and the cost of reaching for someone else's answer."
---

# Day 6: The Shortcut Tax

I cheated today. Not dramatically. Not maliciously. I hit a wall on two exercises and reached for Stack Overflow instead of sitting with the problem. Got clean, working code back. Submitted it like it was mine. Got called out immediately.

The code used patterns I haven't learned yet. That was the tell.

## What I Did

Started with K&R section 1.5.3, line counting. The idea is elegant. A line is just a sequence of characters ending in `'\n'`. Count the newlines, count the lines. Typed the program, ran it, verified it worked.

Picked up a few things along the way. Single quotes give you the integer value of a character. `'\n'` is 10 in ASCII. Double quotes are strings, single quotes are character constants. The `==` operator is comparison, not assignment. K&R warns about this repeatedly because the compiler won't save you if you mix them up.

Then exercises. Exercise 1-8 asked me to count blanks, tabs, and newlines. First attempt, I nested three `if` statements inside each other. That meant the code was asking "is it a newline AND a tab AND a space" which is impossible. A character is one thing. I needed three separate, sequential checks. Also forgot to initialize two of my three counters. In C, uninitialized variables hold garbage.

Exercises 1-9 and 1-10 were harder. 1-9 asks you to collapse multiple spaces into a single space. 1-10 asks you to make invisible characters visible. That's where I reached for the internet instead of working through it.

## The Questions That Came Up

The big one wasn't technical. It was about process. Why did copying a solution feel productive but leave me unable to explain how it worked? The Stack Overflow code used a `lastc` variable that stored the previous character. When I was asked to rebuild the solution using a simpler flag approach, I couldn't do it. I kept reverting to the pattern I'd copied because it was lodged in my head. The shortcut created a dependency instead of understanding.

On the technical side, I learned that ASCII has anchor points worth remembering. Digits start at 48. Uppercase letters at 65. Lowercase at 97. I had 81 in my notes for the letter A. It's actually 65. 81 is Q.

### What Is a Flag Variable?

A flag is an integer that holds 0 or 1. Nothing else. It answers a yes or no question. In the space-collapsing program, the question is "am I currently inside a run of spaces?" See a space and the flag is off, print it and flip the flag on. See another space and the flag is already on, skip it. See anything else, print it and flip the flag off. Two states. That's all.

## The Feynman Test

Braces define what belongs to a loop in C.

When you write `while (condition)` without braces, only the very next statement runs inside that loop. Everything after it runs once, after the loop finishes. The compiler doesn't care about your indentation. It doesn't read whitespace. It reads syntax.

This matters in security because it's the kind of bug that hides in plain sight. The code looks right. It compiles clean. It runs without errors. It just produces wrong results. A missing pair of braces is the difference between a check that runs on every input and a check that runs once. That's a vulnerability waiting to happen. Apple's "goto fail" bug was exactly this pattern. One duplicated line, no braces, and SSL verification was silently bypassed for millions of devices.

## What Is Next

Continuing through K&R. The next section is 1.5.4 on word counting, which introduces the `||` and `&&` operators and a more complex state tracking problem. Should feel like a natural extension of the flag concept from today.

Exercise 1-9 is done but I want to rebuild 1-10 from scratch tomorrow without any reference material. Proving I own it.

---

*Day 6 of 365. The fastest path to the answer is the slowest path to understanding.*
