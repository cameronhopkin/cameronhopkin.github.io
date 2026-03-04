---
title: "Day 013 Lucky Number"
date: 2026-03-02T20:00:00-05:00
draft: false
tags: ["c", "knr", "tabs", "scope", "state-management"]
description: "K&R exercises 1-20 through 1-22. Tab expansion, tab compression, line folding. A bug that hid in plain sight."
---

# Day 13: Lucky Number

Thirteen is my lucky number. Always has been. Today felt like proof. Three exercises done, a real bug caught, and the kind of quiet satisfaction that comes from building things that work correctly for the right reasons.

Section 1.10 covers external variables and scope. The reading went fast. The exercises did not.

## What I Did

Exercises 1-20 through 1-22. All three deal with text transformation, which sounds boring until you realize how much real software does exactly this.

**1-20: detab.** Replace tabs with the right number of spaces. The trick is tracking your column position. A tab does not mean "insert 8 spaces." It means "advance to the next tab stop." If you are at position 3, that is 5 spaces. Position 7, that is 1. The `do-while` felt natural here. You always emit at least one space, then keep going until you hit the boundary.

**1-21: entab.** The inverse. Replace runs of spaces with tabs where they align. This one required an inner loop to consume the spaces, count them, then figure out how many tabs fit. The logic for calculating the first tab width from an arbitrary starting position took some thought. `TABSTOP - (start_pos % TABSTOP)` gives you the distance to the next stop from wherever you are.

**1-22: fold.** Break long lines at word boundaries. This was the most complex of the three. Buffer the line. When you approach the fold point, walk backward looking for a space. If you find one, split there. If you don't, hard break. Then shift the remaining characters back to the start of the buffer and keep going.

The character shifting logic in 1-22 is the kind of thing that feels fragile when you write it. Two index variables walking through the same array at different offsets. I kept it simple and it worked, but I can see how this pattern goes sideways in production code.

## The Questions That Came Up

### The Hidden Newline Bug

Exercise 1-21 had a bug I did not catch on my own. When a run of spaces ends with a newline, my inner loop consumed that newline with `getchar()`. The character fell through to a `putchar(c)` and `pos++` at the bottom of the space handling branch. Output looked fine. But `pos` never reset to zero.

The newline reset lives at the top of the main loop, in the `if (c == '\n')` check. A newline consumed inside the space branch never passes through that check. So `pos` keeps climbing across lines. Every tab stop calculation after the first line is wrong.

The fix was small. Check for `'\n'` at the point where the consumed character gets output:

```c
if (c != EOF) {
    putchar(c);
    if (c == '\n') {
        pos = 0;
    } else {
        pos++;
    }
}
```

Small fix. Big lesson.

## The Feynman Test

Imagine you have a counter that tracks where you are on a page. Like a typewriter carriage position. Every character advances it by one. Every newline resets it to zero.

Now imagine your program has two different doors a character can walk through. Door A checks "is this a newline?" and resets the counter. Door B handles spaces and sometimes accidentally lets a newline slip through without the reset.

The counter keeps climbing. Nothing crashes. The output looks correct at first glance. But every calculation that depends on "where am I on this line" is now wrong. Silently. Invisibly.

That is what happens when the same piece of information, your position on a line, can be modified in multiple places, and one of those places forgets to handle a case. The more places that can touch a variable, the more chances for one of them to get it wrong.

## Hacker Connection

Section 1.10 is about external variables and scope. The reading shows how moving variables outside of functions makes them accessible everywhere. Convenient. Dangerous.

My bug in 1-21 was a miniature version of the same problem. One variable, `pos`, with two code paths that could modify it. One path forgot to handle a case. In a 40-line program I caught it because someone asked the right question. In a 40,000-line codebase with `pos` declared `extern` and six files updating it, that bug lives forever.

Shared mutable state with multiple update paths. That is the vulnerability pattern. It connects to race conditions in concurrent code, to TOCTOU bugs where state changes between check and use, to every global variable in every C program that grew beyond what one person could hold in their head.

The XZ Utils backdoor (CVE-2024-3094) exploited build system state that multiple scripts could modify. The attack hid in the complexity of shared state. When everything can touch everything, auditing becomes impossible and attackers get cover.

## What Is Next

Exercises 1-23 and 1-24 to finish Chapter 1. Then Chapter 2 opens up types, operators, and expressions, which is where integer overflow and type confusion start becoming concrete. The hacker track scales up in Phase 2 with the first deliberately vulnerable program.

Still carrying the sentinel value collision from Day 12. Zero meaning two things. That pattern will come back.

---

*Day 13 of 365. Two doors. Same variable. Only one remembered to reset it.*
