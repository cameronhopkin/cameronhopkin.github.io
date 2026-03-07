---
title: "Day 17: The Break Day That Wasn't"
date: 2026-03-06T00:00:00-05:00
draft: false
tags: ["3d-printing", "break", "tools", "hardware"]
description: "Took a day off from K&R and spent it getting a dead 3D printing lab alive. Turns out debugging firmware and tracing undocumented wiring is just hacking with different stakes."
---

# Day 17: The Break Day That Wasn't

Some days you put down the book. Today was a 3D printing day. My wife and I traded a PS5 back in September as part of a gaming addiction intervention. That trade eventually led to acquiring a complete 3D printing workshop from someone whose uncle had passed away. The uncle was serious. Today we inventoried it, triaged it, and got a printer running.

It was supposed to be a rest day.

## What I Did

Went through every drawer of a multi-compartment parts organizer and a large storage bin, photographing each section and identifying every component. 250-plus nozzles across sizes from 0.2mm to 1.5mm. Mainboards, stepper motors, complete axis assemblies, BL Touch in box, all-metal hotends, upgraded fans. The retail value of what we received sits somewhere above two thousand dollars. We traded a gaming console for it.

My daughter sorted seventy rolls of filament using the bend test. She evaluated every spool for brittleness, set aside the ones sealed in vacuum bags, flagged the ones that needed drying, and tossed four. She is twelve. She did it methodically and without being asked twice. That was the best part of the day.

We got the primary Ender 3 Pro running. 4.2.7 32-bit board, dual-gear extruder pre-installed, copper bed springs, glass bed. Manual bed leveling using a community G-code routine, PETG first print, then PLA. The PLA print was clean. The PETG was rough because the filament had moisture in it despite being from a sealed bag. Good information to have.

Then we tried to install the BL Touch.

The short version: official Creality firmware for the Ender 3 Pro is compiled for a DWIN touchscreen, not a monochrome LCD. Every official download. All of them. The board we have does not have a dedicated 5-pin BL Touch connector on this production run. The green adapter board we had was faulty. The BL Touch probe pin got destroyed when the trigger signal did not reach the board in time.

The fix came from a community build on GitHub. Found via a forum post, fetched it with curl in Terminal, formatted the SD card properly to clear Mac hidden files, and got the screen back. BL Touch self-tests and deploys. The Z-endstop trigger still needs work so we reverted to the manual switch for now. The probe is physically mounted and waiting.

Two calibration cubes printed. Printer confirmed mechanically sound.

## The Questions That Came Up

Why did the official Creality firmware not work? Because Creality ships the same product with two different display types and does not label the downloads clearly. The monochrome LCD and the DWIN touchscreen are not interchangeable at the firmware level. Every download page assumes you have the touchscreen. This is the kind of thing that looks like user error until you read the right forum thread.

Why did the 16GB card fail and the 8GB succeed? SD card compatibility with printer bootloaders is a known edge case. Some bootloaders only negotiate correctly with cards under a certain capacity or formatted with specific cluster sizes. The tool for getting it right on a Mac is `diskutil` and `newfs_msdos`, not the GUI disk utility.

How is the BL Touch supposed to work on a board without the dedicated port? You remap. The servo signal wire goes to a free GPIO pin that the firmware is configured to use instead. In our case that meant the filament sensor header. It works because firmware is software and GPIO is just a pin. The question is always whether the community firmware has that pin mapped correctly.

## The Hacker Connection

Light one today, but it is there.

The official firmware downloads were useless. The correct tooling came from the community, fetched directly, verified against forum documentation, and applied with manual steps instead of the official GUI process. This is exactly the dynamic in vulnerability research. Vendor advisories are often incomplete, sanitized, or lag behind what practitioners already know. The signal is in the forums, the GitHub issues, the IRC logs, the commit history. Learning to find the real documentation instead of the official documentation is a skill, and today was a small rep of that.

Also: firmware flashing is just writing code to a device. The failure modes, the debugging loop, the "why does this not work" cycle, it runs the same way whether the target is a printer board or a microcontroller you are trying to instrument during an engagement. Patience and methodical elimination. No guessing.

## What Is Next

Day 18 gets back to K&R. Section 2.3, character constants. The overnight question that has been waiting: why is `'A'` an `int` in C rather than a `char`. I have a hypothesis. Tomorrow I find out if I am right.

K2 Plus Combo is in transit from Orlando. That will be its own project.

---

*Day 17 of 365. A break day is just a day you debug something different.*
