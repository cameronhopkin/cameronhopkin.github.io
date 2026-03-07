---
title: "Day 18: Why 'A' Is Not A Character"
date: 2026-03-07T00:00:00-05:00
draft: false
tags: ["c", "types", "operators", "escape-sequences", "short-circuit", "buffer-overflow"]
description: "Closed the overnight question on why character constants are int, worked through sections 2.3 through 2.5, and wrote exercise 2-2 — the one where short-circuit evaluation turns out to be a security guarantee."
---

# Day 18: Why 'A' Is Not A Character

Came into today with one question that had been sitting overnight: why is `'A'` an `int` in C instead of a `char`. I had a hypothesis. The hypothesis was partly right and mostly incomplete. That gap got closed today before anything else moved.

## What I Did

Started by working through the overnight question on character constants and int. Then read sections 2.3, 2.4, and 2.5. No exercise in 2.3. Section 2.4 was short. Section 2.5 was precedence rules. Exercise 2-2 closed out the session: rewrite a for loop without using `&&` or `||`.

Exercise 2-2 came from K&R's string-reading loop:

```c
for (i = 0; i < lim-1 && (c=getchar()) != '\n' && c != EOF; ++i)
    s[i] = c;
```

My solution:

```c
i = 0;
while (i < lim - 1) {
    c = getchar();
    if (c == EOF)
        break;
    if (c == '\n')
        break;
    s[i] = c;
    i++;
}
```

The thing that matters here is not the structure. It is the order. `getchar()` only gets called when `i < lim-1` is true. That is not an accident.

## The Questions That Came Up

### Why are there only 13 escape sequences and why are two of them multi-character?

Thirteen covers two categories. The first is control characters that predate modern terminals: bell, backspace, formfeed, carriage return. Characters that did physical things to hardware. The second is characters that would break the parser if you typed them literally: backslash, single quote, double quote. You cannot write a literal backslash inside a string without the parser consuming it as the start of an escape.

`\ooo` and `\xhh` are multi-character because they are not fixed mappings. They are a syntax for specifying any arbitrary byte by its numeric value. Octal came first because the PDP machines C was written on were octal-native. Hex came later in C89 because humans read hex more easily. Both exist because you sometimes need to embed a byte whose value you know but whose printable form does not exist.

### What is actually different between 'x' and "x"?

This one landed harder than expected. `'x'` is an integer. It is the numeric value of the letter x in the machine's character set. On an ASCII machine that is 120. `"x"` is an array of two characters: the letter x followed by a null terminator. One is a number. The other is a small piece of memory with a hidden character at the end.

I have almost certainly used these interchangeably at some point in something I wrote. They are not interchangeable. The null terminator in the string is not decoration. It is load-bearing.

## The Feynman Test

Why is `'A'` an `int` in C and not a `char`?

Because C needs a way to signal end-of-file through the same channel it uses to return characters. `getchar()` reads one character at a time. But it also needs to tell you when there are no more characters. It does this by returning a special value called EOF, which is defined as -1.

If `getchar()` returned a `char`, you would have a problem. On a machine where `char` is unsigned, -1 cannot be stored in a char at all. It wraps around to 255. Now EOF looks like a valid character and your loop never terminates. On a machine where `char` is signed, -1 can be stored. But so can a legitimate byte with that bit pattern. Now a valid input byte looks like EOF and your loop exits early, silently dropping data.

The fix is to make character constants live in the same type as EOF. Both are `int`. The comparison `c != EOF` is then unambiguous regardless of what machine you are on.

This is why `'A'` is an integer. Not because of performance. Because of a signaling problem that would break input handling on half the machines on earth if it were not solved at the type level.

## Hacker Connection

The short-circuit evaluation in exercise 2-2 is a bounds check. The condition `i < lim-1` must be true before `getchar()` is called. If the buffer is full, the character is not read. That ordering is the protection.

This is not just a style choice. It is a security property. The bounds check and the operation it guards must stay in order. If they are separated, if the check becomes unreachable, or if a compiler reorders them under optimization assumptions, you now have a read with no validated destination. Data lands outside the buffer.

Real CVEs follow this exact pattern. A bounds check and a write get separated by a refactor, an inline, or an optimization pass. The check exists in the source. It does not exist in the binary path that matters. Memory gets written past the end of the buffer.

This also connects to the sentinel collision entry already in the vulnerability notebook. The `'x'` vs `"x"` distinction is the same class of problem at a smaller scale: two things look similar, one carries a hidden value, and a program that confuses them has a logic error waiting for the right input.

## What Is Next

Section 2.7 is bitwise operators. I have already used `~` in exercise 2-1 to derive integer ranges by flipping every bit in zero. Tomorrow I find out what `&`, `|`, `^`, and the shift operators do and where they show up in security contexts.

The overnight question: what does `&` do to two integers, and where have you seen that operation used in security work?

---

*Day 18 of 365. A character constant is an integer wearing a costume.*
