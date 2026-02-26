---
title: "Day 002 the Type System"
date: 2026-02-19T08:50:13-05:00
draft: false
tags: []
description: ""
---

# Day 002: The Type System

Today I opened K&R section 1.2 — Variables and Arithmetic Expressions.
A Fahrenheit-to-Celsius conversion table. Fourteen lines of code. It
broke my assumptions about how computers do math.

## What I Did

I typed in the integer version of the temperature conversion program.
Compiled it. Ran it. The output was wrong. Not wrong because of a typo.
Wrong because I told the machine to do integer math and integer math
throws away remainders.

The formula is `celsius = 5/9 * (fahr - 32)`. When both `5` and `9` are
integers, C computes `5/9` as `0`. Not `0.555`. Zero. Integer division
truncates toward zero. Multiply anything by zero and every Celsius value
comes back as zero.

The fix: declare the variables as `float` instead of `int`. Write `5.0/9.0`
instead of `5/9`. Now the machine does floating point math and the table
comes out right.

This is not a quirk. This is the machine telling me exactly what I asked
for. I asked for integer division. I got integer division. C does not
protect you from yourself.

## The Questions That Came Up

I spent the session writing down every question the reading triggered
before looking anything up. Here is what I found.

### What are the basic data types in C?

C gives you types that map to what the hardware provides. `char` is one
byte. `short` is usually two. `int` is usually four. `long` is four or
eight depending on the platform. `float` is four bytes. `double` is eight.

The important part: C does not guarantee sizes. `int` is defined as "at
least 16 bits." On my machine it is 32 bits. On the PDP-11 where C was
born it was 16. The language was designed to run on any hardware. If you
need exact sizes — and in security work you almost always do — you use
types like `int32_t` and `uint8_t` from `stdint.h`.

Rust learned from this. Every integer type has its size in the name:
`i32`, `u8`, `u64`. No ambiguity. Python went the other direction and
hid the machine entirely. Python integers grow as large as your memory
allows. No overflow. No awareness of what is happening underneath.

Both are valid choices. But if you want to understand the machine — and
especially if you want to understand why software breaks — you need to
know what C is doing.

### Why did my compiler reject code without `int main`?

Yesterday my hello program compiled fine. Today I wrote a program without
`int` in front of `main` and the compiler told me "ISO C99 and later do
not allow implicit int."

In original K&R C from 1978, if you left off the return type, the compiler
assumed `int`. This was called implicit int. The C99 standard killed it.
You now have to say what you mean.

K&R the book was written during the transition from the old style to ANSI
C. Some examples still use the old conventions. When my compiler rejects
something from the book, I am watching forty years of language evolution
happen in real time. That is not an obstacle. That is the education.

### Where does printf come from?

`printf` is not part of the C language. It is part of the C Standard
Library — libc — included through the `stdio.h` header.

Here is the actual chain of events when I call `printf("hello")`:

My program calls `printf`. The `printf` function in libc does all the
formatting work — handling `%d`, `%f`, padding, precision — in user space.
Then it calls `write()`, which is a system call. The kernel receives that
call and sends the bytes to my terminal.

`printf` is a convenience layer. I could skip it entirely and call
`write(1, "hello\n", 6)` directly. That day is coming.

Every language seems to have printf-style formatting because every
language that came after C was either written in C, ran on an OS written
in C, or was designed by people who grew up with C. Python's `%`
formatting. Rust's `println!` macro. Java's `printf`. All descendants.

### Format specifiers

The `%` constructions in printf are called format specifiers. The main
ones: `%d` for integers, `%f` for floats, `%c` for characters, `%s` for
strings, `%x` for hexadecimal, `%p` for pointer addresses.

You can add width and precision: `%6d` prints an integer in a field at
least six characters wide. `%6.1f` prints a float in a field six wide
with one decimal place. This is how you make columns line up.

There is also `%n`, which writes the number of characters printed so far
into a variable. It is a legendary source of security vulnerabilities.
Format string attacks exploit `%n` when user input gets passed directly
to printf without sanitization. I will study that in detail later.

### Why are variables declared at the top?

In C89 — the standard K&R second edition targets — declaring all variables
at the beginning of a block was a rule, not a suggestion. The compiler
needed to know all variable sizes up front to set up the stack frame.

C99 relaxed this. You can now declare variables anywhere, including inside
a for loop: `for (int i = 0; i < 10; i++)`. But the book follows C89
conventions, so everything is at the top.

Variables declared inside a function are local to that function. They live
on the stack. When the function returns, that memory is reclaimed. If you
return a pointer to a local variable, that pointer now points at memory
that no longer belongs to you. That is a dangling pointer — one of the
most dangerous bugs in C. We will get there.

## The Feynman Test

How does C decide whether to do integer math or real math?

C looks at the operands. If both are integers, it does integer math and
throws away any remainder. If either operand is a float or double, it
promotes the other to match and does floating point math. The programmer
decides which kind of math happens by choosing the types. The machine
does exactly what you tell it. Nothing more.

Why should a security engineer care? Because integer truncation and
overflow are real vulnerability classes. A calculation that looks correct
in your head can produce zero — or worse, a negative number that wraps
around to a massive positive — when the machine does integer math. Bounds
checks fail. Buffer sizes go wrong. Exploits follow.

Understanding the type system is not academic. It is the first layer of
understanding how software breaks.

## What Is Next

Sections 1.3 and 1.4 — the `for` loop and symbolic constants. The
temperature table rewritten three ways, each one teaching something new
about how C thinks.

Also: twenty minutes of vimtutor. The editor is part of the craft.

---

*Day 2 of 365. The type system is not a convenience. It is a contract
between you and the machine.*


