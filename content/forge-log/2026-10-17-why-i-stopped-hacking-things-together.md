---
title: "Why I Stopped Learning by Hacking Things Together"
date: 2026-10-17T20:00:00-04:00
draft: true
series: ["Forge Log"]
tags: ["forge-log", "python", "learning", "ai", "agents", "faith"]
description: "I sat a Python baseline with no code and no AI, missed two classic traps, and built a coach instead of reaching for another shortcut."
---

Today I sat down with three Python problems, a chat window, and one rule. No running code. No AI help. I wanted to see what was actually in my head before I built anything to coach me.

## The Baseline

Problem one was a function that takes a block of text and returns the top three words. Here is my answer, typos and all:

> would be a sort based on unique count of words that you set lowercase to on first then you just take the top three if there. is a tie base it off alphabetic ordering. what that looks like in any code I am not confident enough.

The design was right. The code never showed up.

Problem two was counting failed logins by IP address from a list of log lines:

> you can do this several ways so again not sure but probably regex to get the IP and and FAIL or OK then do a count on them both and then just print the IP and counts by ip and stdout it.

Right again. No code again. Then I typed this: "See this is why I feel like a fake."

I know what I want the program to do. Getting it onto the screen from memory is where it falls apart.

Problem three was the one I got wrong:

```python
def add(x, bucket=[]):
    bucket.append(x)
    return bucket

print(add(1))
print(add(2))
fns = [lambda: i for i in range(3)]
print([f() for f in fns])
```

My guess:

> I think it prints 1,2, 3 because the range is 3 and it will print add 1 and then add 2 and it appends that to a bucket htat is returned and there is a loop - can be sure though because that last print should print the F so it could actually break

It prints `[1]`, then `[1, 2]`, then `[2, 2, 2]`. Both halves wrong.

Python builds that empty list once, when the function is defined, so every call shares it. The lambdas look up `i` when they are called, and by then the loop is done and `i` is 2. Both traps have their own entry in the official Python FAQ. The one on default values says it plainly: "This type of bug commonly bites neophyte programmers." The details are in [Python Journey Day 001](/python/day-001-predict-then-run/).

## What I Used to Do

I have been doing this for more than twenty years. My method has always been to get something working with the docs or an AI open beside me, and move on. It works. It also means I can ship a thing and still miss a trap that has a name.

I have done this on this site before. On [Day 6](/posts/day-006-the-shortcut-tax/) of the C Journey I hit a wall on two exercises, pulled code off Stack Overflow, and got caught because it used patterns I had not learned yet. Three days later, on [Day 9](/posts/day-009-shortcut-tax-repaid/), I rewrote ten exercises from scratch with nothing open. I made it to Day 30 and stopped. C is still waiting. I will pick K&R back up once I am more comfortable.

Working is not the same as knowing. So I stopped learning by hacking things together, and I built a coach.

## The Coach

The coach is a skill coach I set up in Grok Bot, the coordinator that runs the rest of my AI bot team. It is built on two things: Zed Shaw's book and a skills gap assessment of me. Its instructions are to build me out with security and AI along the way, not just Python.

That assessment is my real record, not a generic beginner profile: what I am strong at, where I have gaps, how I learn, the traps I fall into, and what comes before study time. Every rule the coach runs answers one of those traps.

**I rush.** So every exercise gets typed by hand, and I write down what I think it will print before I run it. Prediction first. No exceptions.

**I start strong and stall.** K&R is the proof. So nothing counts as learned the day I learn it. Each lesson gets a recall check at one, three, seven, and twenty-one days.

**I can explain things I cannot do.** The baseline proved that too. So each lesson gets a Feynman note: three to five plain sentences and one small example. If you read the C Journey, you have seen a Feynman Test section before. A note only counts if I can redo it from memory a week later.

**I lean on whatever is open beside me.** So docs and AI stay open while I learn, then close for a build at the end of each level. Closed book. Just me and the terminal.

**I am not starting from zero.** Zed Shaw's *Learn Python the Hard Way, 5th Edition* is the spine, and I am reading it on his website. The coach goes past the book on purpose. Security questions and AI tests start at lesson zero, not chapter twelve, because that is the work I do.

**First things stay first.** Study fits around prayer and the Word in the morning, church, and family. Not the other way around.

## Six Lessons In

Lessons 0 through 5 are done and in a git repo. I skipped the book's Jupyter start and set up Homebrew Python 3.14, a virtual environment, git, and nvim instead. I got `//` slightly wrong, misnamed a `.gitignore`, and ran a small prompt injection test with the word PINEAPPLE. That is all in [Day 001](/python/day-001-predict-then-run/), the first post in a new section on this site: the Python Journey.

Tracking starts Monday, October 19. Lesson 6 is Wednesday. On Saturday, October 24, I explain f-strings from memory with no notes. That is the first real test of whether any of this stuck.

## Getting and Understanding

> Getting wisdom is the wisest thing you can do! And whatever else you do, develop good judgment. (Proverbs 4:7, NLT)

I have been good at getting things to work. The working code, the passing build, the answer. Good judgment is slower to develop, and I owe it more time.

If you are learning Python too, copy problem three above and write down your guess before you run it. Then check how close you got.

---

*Three problems, two right designs, one wrong guess. Now I build it slow.*
