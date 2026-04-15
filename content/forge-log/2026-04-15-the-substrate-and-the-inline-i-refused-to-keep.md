---
title: "The Substrate, and the Inline I Refused to Keep"
date: 2026-04-15
draft: false
series: ["Forge Log"]
tags: ["shieldcv", "forge-log", "security", "csp", "webcrypto", "indexeddb", "pdfjs", "codex"]
summary: "Three more prompts done on ShieldCV. The crypto and storage substrate is in place at 100% coverage, the resume editor works with sandboxed PDF import, and Codex twice hit the wall and stopped instead of quietly weakening the security posture. Notes on what that looks like in practice."
---

The first Forge Log post ended with the foundation in place: a PWA shell, header-delivered CSP, Trusted Types live, a security posture banner, and a manually verified XSS block in the browser console. That was the day one story.

The next three prompts were supposed to be the easy part. They weren't. They were where the pattern that I think actually matters with AI coding tools showed up twice in a row.

## What got built

Prompt 2 was the crypto package. Zero runtime dependencies. PBKDF2-SHA-256 at six hundred thousand iterations to derive an AES-256 key from the user's passphrase. AES-GCM authenticated encryption with a versioned bundle format so the algorithm can be migrated later without breaking existing data. SHA-256 hex hashing. Constant-time hex comparison for any place where a timing side-channel could leak information. A threat model document checked in next to the code. Thirteen Vitest tests. One hundred percent coverage on every dimension the coverage tool measures.

Prompt 3 was the encrypted storage layer. An IndexedDB wrapper that takes plain JavaScript objects, JSON-encodes them, encrypts them with AES-GCM via the crypto package, and stores the ciphertext in the user's browser. The derived key lives in a hard-private class field with the ECMAScript hash syntax, not the soft TypeScript private that anyone can pierce with `as any`. The lock method nulls the key reference and zeroes any retained backing bytes, which is the best you can do in a JavaScript runtime that doesn't let you reach into raw memory. The passphrase is verified by trying to decrypt a known sentinel value: if AES-GCM authentication passes, the passphrase is right; if it fails, the unlock throws. There's no string comparison anywhere in the verification path, which means there's no timing side-channel to worry about. Seventeen tests. One hundred percent coverage.

The non-negotiable test in that package is the one that proves the encryption actually works. It puts a resume object into storage with a very specific unique string in one of the fields, reads the raw IndexedDB record directly through a separate idb handle that bypasses the encrypted wrapper entirely, serializes the raw record to a string, and asserts that the unique string does not appear anywhere in those bytes. If that test passes, the data is genuinely encrypted at rest. It passes.

Prompt 4 was the resume CRUD experience. A new package for the canonical resume model based on a subset of the JSON Resume schema with Zod runtime validation. Three new routes: a list view, a new-resume bootstrap, and an editor. An inline vault unlock panel that appears on every protected route so the user can unlock without bouncing through a separate redirect. Encrypted persistence under the resume namespace via the storage package. Debounced autosave with a three-state indicator: Saving, Saved, Save failed with retry. A styled native dialog for delete confirmation, sharing the same component pattern across the list and editor pages. PDF import via a sandboxed iframe at a separate route running pdf.js, parsing the extracted text into the resume model with simple section heuristics. A Playwright end-to-end test that walks the full create-edit-delete flow.

That's the substrate. Backend-equivalent code that runs in the browser, with the encryption and tamper-evidence properties that a backend security team would want from a real database, but no database. No server. No cloud.

## The first wall

Codex hit a real architectural wall during Prompt 1's cleanup work. The default Trusted Types policy needed to be established before any other script ran, and it was sitting in an inline script tag at the top of the HTML head. The strict CSP that I had asked for explicitly forbade inline scripts. So either I had to relax the CSP, or I had to find another way.

I asked Codex to investigate whether SvelteKit's static adapter could produce a build that satisfied all three constraints I cared about: header-delivered CSP, no meta tag fallback, zero inline scripts. I told it to stop and tell me if it hit a wall.

It hit a wall, honestly. It came back and said something like: "with the current static adapter, I cannot produce a build that is both fully functional and meets your header-only CSP requirement without the meta-tag tradeoff." Then it offered three architectural options. Switch to a server adapter so per-request nonces become possible. Accept the meta-tag CSP and document the tradeoff. Redesign the app to avoid client-side hydration entirely.

That stop was the right call. The temptation in the AI tool world is to reward the assistant that always finds a way. But the assistant that finds a way by silently weakening the thing you asked it to protect is the assistant that ships you a vulnerability you don't know about until someone else finds it. The assistant that stops at the wall and tells you the truth is the one you can actually trust to handle security-sensitive code.

We picked option one. SvelteKit on the Cloudflare adapter, where every request flows through a thin worker that generates a fresh random nonce per request and sets the CSP header on the response with that nonce inside `script-src` alongside `strict-dynamic`. The other security headers get set by a `hooks.server.ts` handler. The worker never touches user data. ShieldCV stays local-first because all resume processing happens in the browser, in encrypted IndexedDB, after the page has finished loading. The worker exists purely to harden the delivery layer. There is no server in the sense that matters: no database, no persistent state, no resume bytes ever crossing the network boundary after the initial load.

That decision unlocked everything that followed. Per-request nonces are the gold-standard CSP pattern. `frame-ancestors`, `report-uri`, and `report-to` all work because the policy is delivered in the response header where they're respected. The Prompt 9 attack mode dashboard, which is going to stream live CSP violation reports during the demo, depends on this entirely. None of it works with meta-tag CSP.

## The second wall

Then it happened again on Prompt 4.

The PDF import requirement was that pdf.js would only run inside an isolated route at `/pdf-worker`, served as a sandboxed iframe with its own CSP. The main app would never import pdf.js, never load it, never give it permission to execute. The parent route would post the file bytes to the iframe via `postMessage`, the iframe would parse the PDF and post back extracted text, and the parent would convert the text into a resume structure with simple heuristics.

Codex built this. The pdf.js dependency stayed isolated to the worker route. The path-specific CSP machinery in `security.ts` correctly served different headers for `/pdf-worker` than for the rest of the site. The architecture was right.

Except the worker route's CSP had `'unsafe-inline'` in `script-src`.

The reason was honest: pdf.js's runtime emits inline scripts during initialization. The fastest path to a working PDF parser was to allow inline scripts on that one route, gated by a sandboxed iframe and `connect-src 'none'`, which technically limits the blast radius. It would have worked. The PDF import would have functioned. Most people would have shipped it.

But the entire ShieldCV pitch is that this is a security-themed product, and `'unsafe-inline'` is the single most-recognizable tell of a CSP that wasn't taken seriously. Anyone with a security background who pulls up the response headers and sees `'unsafe-inline'` anywhere on any route is going to draw the same conclusion: the developer compromised when it got hard. Everything else on the site instantly becomes suspect, even if it's actually rigorous.

So I told Codex to remove it and try the per-route nonce mechanism instead. Same SvelteKit feature we used on the main app. If that didn't work, fall back to moving pdf.js's initialization into an external module file. If that didn't work, document the constraint in code and try one more approach. Don't silently keep `'unsafe-inline'`.

It worked. The current state of `security.ts` has zero `'unsafe-inline'` directives anywhere on the site. The PDF worker uses a SvelteKit-issued nonce for its boot script and a narrow style hash for the one inline style attribute SvelteKit emits. A judge inspecting the headers gets the same answer on every route: this app does not authorize inline scripts.

## The competition is showing me what the alternative looks like

A few people in the Discord for the challenge have posted their submissions. Most of them are competent. Some of them are slick. One of them has a pricing page already, which is a strange choice for a hackathon submission.

What every one of them has in common, as far as I can tell from the outside, is that the resume goes into a cloud LLM. The privacy policies, where they exist, are vague about retention. Most require a Google or Microsoft sign-in before you can see a single feature. None of them, that I have found, can show you their data flow diagram and have it terminate cleanly at the browser boundary. None of them can let you press F12, inspect the network tab during a full session, and verify zero external requests to a server they control.

I don't have a problem with their submissions. They're all in the dominant pattern of the AI resume tool category, which is the pattern the eighteen tools I researched before this build also follow. The cloud is fast, the cloud is easy, and the cloud has the LLMs people actually want to use.

But the dominant pattern is what created the problem ShieldCV exists to solve. UC Berkeley telling students to redact their personal information before using these tools is not a compliment to the tools. It's a damning indictment that even a top-tier career services office doesn't trust them. There is room for one tool in this market that takes the opposite approach. ShieldCV is trying to be that tool.

The substrate I just finished building is what makes that claim defensible. The encryption is real and tested. The local-first promise is verifiable from the network trace. The CSP is strict and uniform. The Trusted Types policy blocks a real XSS payload from the browser console. None of that requires you to take my word for it. You can clone the repo and run it.

## What's next

Prompt 5 is in flight as I write this. The AI package. Transformers.js running NER and embeddings inside a Web Worker, in the browser, on the user's device. The headline question is whether to host the quantized model files inside the repo or pull them from huggingface.co on first load. The first option keeps the network trace pristine. The second option keeps the repo small. There's a defensible answer to either, and the decision is going to depend on what total model size we land on.

After that, the HIPAA PHI scanner gets built on top of the AI package. Then the GDPR rights tracker and CMMC awareness module. Then the hash-chain audit log. Then the attack mode demo, which is the centerpiece of the live pitch. Then the production deployment with a signed SBOM, a completed DPIA, and the threat model written up for everything that's in the repo.

Eleven prompts total. Five done. Six to go. Twelve days left until the submission deadline. The substrate is real, the architecture is right, and Codex has now stopped at the wall twice when it could have shipped a quietly weaker version. That last property is the one I keep coming back to. The hardest part of building security-sensitive software with AI tools is not getting the AI to write the code. It's catching the moments where it tries to make the constraint feel less inconvenient than you said it should be, and holding the line.

Both walls became architectural pivots that made the product stronger. The Cloudflare adapter switch unlocked per-request nonces, working `frame-ancestors`, and the live CSP reporting that Prompt 9 needs. The PDF worker nonce let us keep the same security promise on every route on the site. Neither pivot was on the original plan, and both of them happened because Codex told me the truth instead of working around the constraint.

The repo is at [github.com/WaypointCA/shieldcv](https://github.com/WaypointCA/shieldcv). It is public. The substrate is in. The features ride on top of it from here.

DPE. GSD. One prompt at a time.
