---
title: "Day 023: The Postmortem"
date: 2026-03-19T00:00:00-05:00
draft: false
tags: ["c", "security", "precedence", "undefined-behavior", "audit"]
description: "Chapter 2 closes. Not with new material but with a live audit that forced application of everything the chapter built."
---

# Day 023: The Postmortem

Chapter 2 is closed. Not just finished. Closed.

Today was not new material. It was the part that most people skip: going back to what you think you know and pressing on it until it either holds or breaks. K&R 2.12 first, then a four-line code fragment that turned out to be a trap with four distinct vulnerabilities stacked inside it. The chapter held. I did not walk out unscathed.

## What I Did

Section 2.12 is the operator precedence table. It is easy to treat it as a reference card and move on. I did not.

The important thing K&R says in 2.12 is that precedence and order of evaluation are independent. Precedence is about parsing: which operands belong to which operators. Order of evaluation is about execution: when does the machine actually compute each operand's value. The table governs the first. It says nothing about the second.

That distinction matters because the whole chapter has been building toward it. The sequence point work from Day 20 lives here. The `i++` and `a[i] = i++` conversations live here. Precedence told us how to read the expression. It never told us when the side effect fires.

We built a full annotated precedence table as a permanent reference — every row flagged with the security trap it carries. It is below. I plan to keep it close.

After the table we ran a live audit. Four lines:

```c
unsigned char buf[256];
int i = 127;
int result;

result = buf[i] & 0xFF == 0 ? i++ : -i;
```

I found five problems. Four notebook entries came out of it.

## The Questions That Came Up

**What is the actual bug in that expression?**

The real bug is precedence. `==` has higher precedence than `&`. So `buf[i] & 0xFF == 0` parses as `buf[i] & (0xFF == 0)`. The comparison `0xFF == 0` is false, which in C is zero. The whole expression becomes `buf[i] & 0`. Always zero. Always false. The ternary condition is permanently broken.

Because the condition never fires, `result` is always `-i`, which is always `-127`. That is not a theoretical risk. It is guaranteed on every execution of that fragment. A negative index passed to an array computes an address before the buffer. The Negative Index Pivot (Entry 19) is not a possibility here. It is the only outcome.

Two bugs chain. The first one guarantees the second one.

**Is `f(i, i++)` undefined behavior because of unspecified evaluation order?**

No. I said yes at first. I was wrong.

Unspecified evaluation order means the compiler can evaluate arguments in any sequence it wants. That is unspecified behavior, not undefined behavior. It becomes undefined behavior only when there is an unsequenced modification: when a variable is both modified and read (or modified twice) between the same two sequence points, for a purpose unrelated to determining the new value.

`f(i, i+1)` has unspecified order but no side effect. Both arguments read `i` but neither modifies it. The result is the same regardless of which argument is evaluated first. Legal, if unpredictable.

`f(i, i++)` has unspecified order and a side effect. `i++` modifies `i`. The read of `i` for the first argument is unsequenced relative to that modification. That is the condition C11 calls undefined. The compiler is not choosing between orderings. It is permitted to assume the state cannot exist and generate anything it wants.

The line is the modification. Not the order.

## The Feynman Test

The distinction between unspecified and undefined behavior is the one I got wrong today, so that is what I want to make sure I actually own.

Unspecified behavior: the standard allows multiple outcomes and does not say which one you get. The compiler picks. The result may vary by platform or optimization level. The program is still legal. Think of it as the compiler choosing which lane to drive in — either lane gets you there, and the standard just does not say which one it will pick.

Undefined behavior is different in kind, not just degree. When you write `f(i, i++)`, you are modifying `i` and reading `i` with no sequence point between them. The C standard does not say "the compiler picks an order." It says the assumption embedded in the standard is that well-formed programs do not contain this. The compiler is permitted to assume *you* will never write it. If you do, the standard makes no promises about what the resulting binary does — not just "unpredictable output" but the compiler may legally eliminate code, reorder operations, or produce instructions that have nothing to do with what you wrote.

The security implication is the part that took me a while to fully absorb: a compiler optimizing around undefined behavior is not doing something wrong. It is doing exactly what the standard permits. The undefined state is mine. I put it there.

## The Precedence Table

K&R §2.12. Higher row binds tighter. Operators in the same row have equal precedence.

| # | Operators | Associativity | Notes |
|---|-----------|---------------|-------|
| 1 | `()` `[]` `->` `.` | left to right | Function call, subscript, member access. `->` and `.` covered Ch. 5–6. |
| 2 | `!` `~` `++` `--` `(type)` `*` `&` `sizeof` | **right to left** | Unary. `*` (deref) and `&` (address-of) covered Ch. 5. ⚠ `++`/`--` side effects: evaluation order unspecified. |
| 3 | `*` `/` `%` | left to right | Multiplicative. ⚠ Integer division truncates toward zero; `%` with negatives is impl-defined pre-C99. |
| 4 | `+` `-` | left to right | Additive (binary). ⚠ Signed overflow is undefined behavior. |
| 5 | `<<` `>>` | left to right | Bitwise shift. ⚠ Left shift into sign bit: UB. Right shift of signed: impl-defined. |
| 6 | `<` `<=` `>` `>=` | left to right | Relational. Result is 0 or 1. |
| 7 | `==` `!=` | left to right | Equality — lower precedence than relational. ⚠ `a < b == c < d` groups as `(a<b) == (c<d)`, not a range check. |
| 8 | `&` | left to right | Bitwise AND — lower than `==`. ⚠ `x & mask == 0` parses as `x & (mask==0)`. Always parenthesize bitwise subexpressions. |
| 9 | `^` | left to right | Bitwise XOR. |
| 10 | `\|` | left to right | Bitwise OR. |
| 11 | `&&` | left to right | Logical AND — short-circuits. Left-to-right order is guaranteed (sequence point). |
| 12 | `\|\|` | left to right | Logical OR — short-circuits. Same guarantee. |
| 13 | `?:` | **right to left** | Conditional. ⚠ Both arms must have compatible types; implicit conversion can hide truncation. |
| 14 | `=` `+=` `-=` `*=` `/=` `%=` `&=` `^=` `\|=` `<<=` `>>=` | **right to left** | Assignment. ⚠ Modifying a variable and using it elsewhere in the same expression without a sequence point: UB. |
| 15 | `,` | left to right | Comma — evaluates left, discards, returns right. Rarely seen outside `for` loops. |

The warning K&R buries at the end of 2.12: the table controls parsing. It says nothing about when operands are evaluated. Function argument evaluation order is unspecified. Between sequence points, modifying a variable twice is undefined behavior regardless of what this table says.

Rows 11 and 12 are the only rows where evaluation order is actually guaranteed.

## Hacker Connection

The audit fragment demonstrates how vulnerabilities compound. Entry 18 (Precedence-Mask Trap) guarantees Entry 19 (Negative Index Pivot). Neither is dangerous in isolation here. Together they produce a deterministic out-of-bounds read on every execution, not a probabilistic one.

This is how real CVEs work. The advisory lists one vulnerability class. The root cause is often two logic errors that interact. A bounds check that is written but mathematically incapable of passing looks like a defended path. It is not. It is a permanently open door with a sign on it that says LOCKED.

CWE-480 (Use of Incorrect Operator) is the precedence trap. It shows up in authentication code more than anywhere else. Permissions flags checked with `==` instead of inside parentheses, or a bitwise test where the comparison fires first. The check compiles clean. The check does nothing.

The vulnerability notebook is growing alongside these posts. When it is mature enough to be useful to someone else, it goes public too.

## What Is Next

Chapter 3: Control Flow.

The overnight question I sat with: what happens in a `switch` when you omit `break`? I reasoned through it before opening the book. Fall-through. The CPU does not re-evaluate the condition. It continues executing instructions for the next case because `case` labels in C are just addresses, and nothing stops execution at a label. The `break` is the escape. Without it, the instruction pointer keeps moving.

Chapter 3 will confirm or correct that. It will also introduce the dangling else problem — a different flavor of the same issue: the structure that looks obvious to a human is not what the parser sees.

---

*Day 23 of 365. Chapter 2 is in the vault. Not because I finished it. Because I couldn't break it.*
