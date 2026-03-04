---
title: "Day 014 The Parser's Dilemma"
date: 2026-03-03T20:00:00-05:00
draft: false
tags: ["c", "knr", "parsing", "state-machines", "injection"]
description: "Finishing Chapter 1 with the two hardest exercises. Comment stripping, bracket matching, and why parsers that don't understand context create entire vulnerability classes."
---

# Day 14: The Parser's Dilemma

Fourteen days. Chapter 1 complete. The last two exercises were harder than anything else in the chapter by a wide margin. Not because the C was more complex. Because the problem required thinking about multiple states at once.

## What I Did

Exercises 1-23 and 1-24. The capstone problems of Chapter 1.

Exercise 1-23 asks you to strip all comments from a C program. Sounds straightforward. Scan for `/*`, skip everything until `*/`, done. Except it is not that simple. The sequence `/*` inside a string literal like `"/* not a comment */"` is data, not a comment delimiter. A comment stripper that does not understand strings will eat your string contents.

Exercise 1-24 asks you to check for unmatched braces, brackets, and parentheses. Same trap in reverse. A `{` inside a comment is not code. A `'` inside a string is not a character constant boundary. The bracket checker has to understand comments, strings, and escape sequences or it gives false results.

Both programs needed the same core structure. A state machine tracking whether I am currently inside normal code, inside a block comment, inside a string, or inside a character constant. The transitions between states have to be precise or everything breaks.

The first version of 1-23 had a bug with lookahead. When scanning for the end of a comment, the sequence `**/` requires care. The inner loop sees `*`, reads ahead, gets another `*` instead of `/`. If you just discard that second `*` and loop, you miss that it could be the start of the `*/` closing sequence. The fix was `c = d; continue;` to push the unconsumed character back into the loop variable without losing it. That pattern showed up in three places across both programs.

The second bug was double output. Inside a string, when handling an escape like `\"`, I printed the backslash, read the escaped character and printed it, then fell through to a `putchar(c)` at the bottom of the block. The escaped quote got printed twice. A `continue` fixed the double print but created a new problem. The `c = getchar()` at the bottom of the loop also got skipped, so `c` still held the old value on the next iteration. Had to add another `getchar()` before the `continue` to properly advance.

The third bug was in 1-24. The original version used `d` for comment-end detection but `d` only got assigned inside a short-circuit evaluation. If `c` was not `/`, the `getchar()` inside `&&` never fired, and `d` held whatever stale value it had from last time. Restructured to use the same pre-read loop pattern from 1-23 and the bug disappeared.

## The Questions That Came Up

### Why does the pre-read loop pattern work so well for parsers?

The standard `while ((c = getchar()) != EOF)` pattern reads at the top. But when you need lookahead, you sometimes consume a character that belongs to the next iteration. The pattern of reading before the loop, reading at the bottom, and using `c = d; continue` when you need to "unread" a character gives you control over exactly when and how the next character enters the loop. It is manual but explicit. Nothing gets lost.

### Why separate `in_string` and `in_char` instead of one `in_quoted` flag?

At first I thought one flag would be enough since the logic is similar. But strings and character constants have different closing delimiters. A `"` does not close a `'` context. Using `current_quote` to track which delimiter opened the context solved this cleanly. The flags are separate for clarity but `current_quote` does the real work.

## The Feynman Test

Imagine you are editing a document and you want to remove all the comments someone left in the margins. Simple enough. But some of the actual text in the document contains the words "see comment on page 12." If you just delete everything that looks like a comment reference, you will destroy real content.

A program faces the same problem. The characters `/*` mean "start of comment" in code, but mean nothing special inside a quoted string. The program has to track where it is. Am I in code right now, or am I inside quotes? The answer changes what the same characters mean.

This is context. The same sequence of characters has completely different meaning depending on what state the program is in when it encounters them. Getting this wrong does not just produce incorrect output. It produces output that looks correct until it hits the one edge case nobody tested.

## Hacker Connection

Parser state confusion. That is the name for what these exercises taught me to handle correctly.

A comment stripper that does not understand strings will corrupt data. A bracket checker that does not understand comments will report false errors. These are annoyances in a classroom exercise. In production software they are vulnerability classes.

SQL injection works because a database query parser fails to distinguish between data and control. The attacker's input contains a `'` that the parser interprets as a string boundary instead of a data character. The parser's state machine has a flaw. It does not correctly track whether it is inside user-supplied data or SQL syntax.

Cross-site scripting is the same fundamental bug in HTML. The browser's parser encounters `<script>` inside what should be a data context and interprets it as a control instruction. State confusion between data and markup.

WAF bypasses exploit disagreements between parsers. The firewall's parser and the application's parser track state differently. The firewall sees safe data. The application sees executable control. Same bytes, different interpretation, because the state machines diverge.

Every injection vulnerability I have investigated in 21 years traces back to this. A parser encountered a delimiter or control sequence and did not correctly determine whether it was in a context where that sequence should be interpreted or ignored. Exercises 1-23 and 1-24 are the smallest possible version of that problem.

## What Is Next

Chapter 1 is done. Twenty-two exercises from hello world to multi-state parsers. Chapter 2 starts tomorrow. Types, operators, expressions. The section on data types and sizes will connect directly to integer overflow vulnerabilities. The section on type conversions will explain an entire class of bugs that show up in CVEs every year.

The vulnerability pattern notebook gets a new entry today. Parser state confusion. When a program interprets data as control or control as data. The root cause of injection attacks across every protocol and language.

---

*Day 14 of 365. The same characters mean different things depending on where you are. Parsers that forget this create the bugs. Attackers that remember it write the exploits.*
