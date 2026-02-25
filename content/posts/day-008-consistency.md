---
title: "Day 008: Consistency"
date: 2026-02-25
draft: false
tags: ["learning-the-hard-way", "dotfiles", "environment"]
---

There is a difference between having tools and having a workshop.

I have four machines. A personal laptop. Two Linux boxes. A work computer. Until today each one felt like a different room. Different shell. Different prompt. Different git identity. I would sit down and spend the first five minutes figuring out where I was and what was configured.

Today I fixed that.

## What I Did

One dotfiles repo now controls every machine. Same shell config. Same aliases. Same editor. Same prompt with the hostname showing so I always know which terminal I am looking at.

The trick was separating what is shared from what is not. Everything universal lives in the repo. Everything specific to one machine lives in a local file that never gets tracked. Signing keys, work email, tool chains that only exist on one box. The shared layer stays identical. The local layer stays private.

The work machine was the hardest. It had years of accumulated config hiding in places I did not expect. A redirect file pointing the shell to a completely different directory. An old prompt theme loading before mine ever got a chance. The goal was not to replace any of it. The goal was to wrap it. Keep everything that works. Layer the new system on top.

Four machines. One push. One pull. Same desk everywhere.

## Questions That Came Up

Why does git let you set a signing key path that does not exist on the current machine? There is no validation at config time. It just fails when you try to commit. That is how a config pushed from one machine silently broke another.

## The Feynman Test

Think of it like packing the same go bag for every trip. You want the same tools in the same pockets no matter which bag you grab. But some things change by destination. The passport. The currency. The adapter plug. Those go in a pouch that stays with that bag. The rest is universal.

That is what dotfiles with local overrides are. The universal layer is always the same everywhere. The local layer never leaves the machine it belongs to.

## Hacker Connection

When your signing key path points to a file that does not exist, your commits go unsigned. When your git email is wrong, your commits show up under the wrong identity. When your prompt does not show the hostname, you run commands on the wrong machine.

Environment consistency is operational discipline. The same discipline behind configuration management in production. Same problem, smaller scale.

## What Is Next

Retype exercises 1-3 through 1-12 on the Linux box from memory. Push them to git. Then section 1.6 on arrays. The workshop is built. Time to use it.

---

*Day 8 of 365. Same desk, four machines.*
