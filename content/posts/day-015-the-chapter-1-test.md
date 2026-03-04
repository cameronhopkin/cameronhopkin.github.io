---
title: "Day 015 The Chapter 1 Test"
date: 2026-03-04T06:00:00-05:00
draft: false
tags: ["c", "knr", "review", "vulnerability-patterns", "precision"]
description: "Testing everything from Chapter 1. Feynman gauntlet, code tracing, security connections, and closing gaps I did not know I had."
---

# Day 015: The Chapter 1 Test

Chapter 1 is done. Fourteen days of typing, compiling, breaking things, rebuilding things I broke by taking shortcuts, and writing about all of it. But finishing is not the same as understanding. Today I stopped moving forward and asked the harder question. Do I actually know this?

## What I Did

Three part test. No compiler. No book. No searching. Just what is in my head.

Part one was a Feynman gauntlet. Five concepts from Chapter 1 explained in plain language. Why getchar returns an int. The difference between = and ==. State machines. The null terminator. External versus local variables. I passed most of them but the precision was not where I wanted it. I said char holds 0 to 255. That is unsigned char. On most x86 systems, plain char is signed, which means the range is -128 to 127. That matters because a byte with value 255 stored in a signed char becomes -1. That is EOF. The collision is not theoretical. It is the default behavior on Minion1.

Part two was code reading. Seven snippets. Trace the output or find the bug. I got all seven right. Integer division truncation. Precedence traps. Uninitialized variables. The missing braces illusion. The one flag was that I ran code on C7 instead of tracing it mentally. I need to trust the model in my head and not reach for the compiler as a crutch.

Part three was the security bridge. Connect five C concepts to vulnerability classes. Arrays with no bounds checking. Format string trust. The copy versus getline design philosophy. Global mutable state. Sentinel value collisions. I got the categories right but my format string explanation was too generic. The real issue is not missing sanitization. The real issue is data crossing into the control plane.

Then we closed the gaps. Four of them. Signed versus unsigned char ranges and why the same program behaves differently on ARM versus x86. Assignment as an expression inside conditions, where = always assigns and the result determines truth. Over-read versus overflow, where one gives the attacker binoculars and the other gives them a pen. And the root cause that format string attacks, SQL injection, and command injection all share: user data gets interpreted as instructions.

## The Questions That Came Up

### When = sits inside an if, does C know the difference between a mistake and a technique?

No. It does not. `if (x = 0)` assigns 0 to x, evaluates to 0, and the body never runs. `if (x = 5)` assigns 5 to x, evaluates to 5, and the body always runs. Neither is a syntax error. The language cannot tell whether you meant to assign or compare. This is why K&R's getchar idiom uses parentheses and != together. Assignment in conditions is legal and intentional there. The same syntax used carelessly is one of the most common bugs in C history.

### Does the same C program really behave differently on my two machines?

Yes. Plain char is signed on x86 (Minion1) and unsigned on ARM (Main). A byte with value 255 is -1 on one and 255 on the other. Same source code. Same intent. Different behavior. This is not a contrived example. It is what happens when someone pastes text with accented characters into a program that reads char instead of int.

## The Feynman Test

When user supplied data is directly mixed with executable code, an attacker can craft input that gets interpreted as instructions rather than data. This breaks the separation between what a program should do and the data it operates on.

That single sentence is the root cause of format string vulnerabilities, SQL injection, command injection, and cross site scripting. Different languages. Different attack surfaces. Same mistake. Data crossed into the control plane.

A format string attack happens when a program passes user input as the first argument to printf instead of as the second. The first argument is not data. It is a set of instructions that tell printf how to behave, how many values to read off the stack, what types they are. Give the user that power and they are not just reading output. They are driving the machine.

## Hacker Connection

Four vulnerability patterns got added or refined in the notebook today.

Signed and unsigned char divergence. Platform dependent behavior from the same source code. This creates bugs that pass testing on one architecture and fail on another. An attacker who knows your deployment target can craft inputs that exploit the specific signed or unsigned behavior of that platform.

Over-read versus overflow. A function that reads without bounds leaks information. A function that writes without bounds corrupts memory. Heartbleed was an over-read. The attacker sent a crafted TLS heartbeat request with a length field larger than the actual payload. OpenSSL read past the end of the buffer and returned whatever was adjacent in memory. Passwords, session keys, private keys. All leaked because one function trusted a length value it should have verified. Same root cause as a buffer overflow. Completely different impact.

Assignment in conditions. `if (auth = 1)` always grants access. `if (auth = 0)` always denies it. Neither checks anything. Both compile without error. In a security context this turns authentication and authorization checks into unconditional grants or denials depending entirely on which value the programmer accidentally assigned.

Data versus control confusion. The unifying pattern behind the most exploited vulnerability classes in software history. If your program cannot tell the difference between an instruction and a piece of data, an attacker will use that confusion to make your program do whatever they want.

## What Is Next

Chapter 2. Types, operators, and expressions. The type system is about to get formal treatment. Integer promotion rules, implicit conversions, and the full precedence table. Every one maps to a vulnerability class. The overnight question: when two different types meet in an expression, which one wins? I think the smaller type gets promoted up. Truncation going the other direction is where the danger lives. Tomorrow we find out.

---

*Day 15 of 365. Finishing a chapter is motion. Knowing what you got wrong is progress.*
