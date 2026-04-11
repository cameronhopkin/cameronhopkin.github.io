---
title: "Day 1: Foundation, False Starts, and a CSP That Actually Works"
date: 2026-04-11
tags: ["shieldcv", "forge-log", "security", "sveltekit", "csp", "codex"]
series: ["Forge Log"]
---

![Tyrminal Forge](/images/tyrminal_forge.png)

I am building a thing called ShieldCV for the Handshake x OpenAI Codex Creator Challenge, and I want to write about it as I go.

Not a launch post. Not a retrospective. A build log. The kind of thing I wish more people wrote when they were in the middle of shipping something hard, because the tidy version that gets published after the fact never tells you what actually happened.

So here is what actually happened on day one.

## What ShieldCV is

A local-first, mobile-first, compliance-aware resume security platform. Your resume never leaves your device. The AI runs in your browser. Encrypted storage with a passphrase you control. HIPAA scanning for healthcare students who put patient details in clinical descriptions without realizing it. CMMC awareness for defense contractor applicants. GDPR rights tracking for international students who can legally request their data back from every job board they have ever used.

I researched the AI resume tool market for a couple of days before this. Eighteen tools. Every single one uploads your resume to the cloud. Most of them keep it for thirty to a hundred and eighty days after you delete your account. UC Berkeley literally tells students to strip their personal information out of their resumes before using these tools. That is the gap. ShieldCV sits in that gap.

The build is happening through OpenAI Codex on a roughly twenty day runway. The challenge ends April 30. Today was day one.

## Prompt 0: scaffold

The first thing I did was lock down a context block that I am pasting at the start of every Codex session. Seven core principles that cannot be violated. Zero data retention. Local-first AI. Encrypted at rest with PBKDF2 at six hundred thousand iterations. Mobile-first. Defense in depth with strict CSP, Trusted Types, sandboxed iframes, and a Permissions Policy that denies camera, microphone, geolocation, payment, USB, and the rest of the surface area browsers shouldn't be quietly handing out. Auditability through hash-chained logs. Compliance artifacts as first-class deliverables: DPIA, threat model, signed SBOM.

That context block is the anchor. Every prompt rides on top of it. Without it, Codex drifts toward whatever pattern is statistically most common in its training data, which for a privacy-themed project means it will helpfully suggest analytics, error tracking, cloud LLM APIs, and "for development convenience" relaxations of CSP. The context block is the thing that tells it no.

Prompt 0 was the scaffold. Pnpm monorepo, SvelteKit web app, five workspace packages stubbed out for crypto, storage, AI, compliance, and audit, GitHub Actions workflows for CI and security scanning, Tailwind, Vitest, vite-plugin-pwa, the whole foundation. Codex shipped it in one pass and all five local checks passed on the first run.

Then I pushed it to GitHub and CI immediately failed.

## The first stumble

Two failures, both Codex's fault, both fixable in one tight follow-up. The CI workflow had pnpm declared in two places that disagreed with each other. The security workflow referenced a GitHub Action path that does not actually exist. Codex had hallucinated `google/osv-scanner-action@v1` when the real action lives at a different path. The fix was a five line change to point at the correct reusable workflow and consolidate the pnpm version into the `packageManager` field where it belongs.

Pushed again. CI passed. Security workflow started running OSV-Scanner against the dependency tree and immediately failed because the SvelteKit toolchain Codex pinned had fourteen known vulnerabilities across SvelteKit, Svelte, Vite, esbuild, cookie, and serialize-javascript.

This is the moment where most people would shrug and move on. The vulnerabilities are in transitive dependencies. None of them are in code I wrote. They are mostly in SvelteKit's own internals, which means they will get fixed when SvelteKit ships the next version anyway. The pragmatic call is to suppress the scanner and keep building.

But ShieldCV's entire pitch is that I take security seriously enough to ship a hardened CI pipeline. If a judge clones the repo and sees a red OSV-Scanner badge, the pitch is over before it starts. So I spent the next hour pushing Codex through a coordinated major version upgrade across the whole web stack. SvelteKit 2.5 to 2.57. Svelte 4 to Svelte 5. Vite 5 to Vite 6. Vite-plugin-svelte to a version that supports both Vite 6 and Svelte 5 without overshooting into a Vite 8 dependency that does not exist yet.

The Svelte 4 to 5 jump is normally a real migration with breaking changes. The placeholder code Codex had written so far did not use any Svelte 4 specific patterns, so the migration cost was zero lines of code. That is the cheapest possible time to take a major version bump. Day one of a build, before any real components exist. If I had waited until day five, this would have eaten an afternoon.

Three transitive vulnerabilities still remained after the upgrade. Cookie and two CVEs in serialize-javascript, both pulled in by deeper layers of the toolchain that had not yet caught up. The fix was a pnpm `overrides` block in the root `package.json` that forces those packages to fixed versions everywhere they appear in the dependency graph. Six lines of JSON.

Pushed. OSV-Scanner came back clean. Zero known vulnerabilities. That is the baseline I want to be standing on when I start building real features.

## Prompt 1: the visible product

This is where things get interesting.

Prompt 1 was supposed to deliver a mobile-first PWA shell with a bottom tab bar, a Security Posture banner showing four enforcement indicators, four interactive modals explaining each one, a `/security` route rendering a docs page, a Trusted Types policy in the head of the document, a CSP violation reporter wired into a Svelte store ready for the eventual attack mode dashboard, and the full security header stack in a `_headers` file for Cloudflare Pages to serve. Big prompt. About fifteen new files.

Codex shipped it. Build passed, all five checks passed. I ran the dev server and the app loaded with a clean dark theme, the navigation worked, the security posture cards rendered with the right copy, and everything looked great.

Then I clicked one of the cards and nothing happened.

I went to the DevTools console and saw a CSP violation. The Trusted Types policy was being delivered as an inline `<script>` block in the document head, and the strict CSP I had asked for explicitly forbade inline scripts. The policy was being blocked from initializing, the modals had no event handlers attached because Svelte's hydration was failing for the same reason, and the whole interaction layer was dead in the water.

This is exactly the situation where most builds quietly slip toward the easy fix. The easy fix is `'unsafe-inline'`, which silences the error and breaks Trusted Types simultaneously by allowing arbitrary inline script execution. I was not going to do that. The right fix is to externalize the Trusted Types policy into a real `.js` file served from `'self'`, which passes CSP without any relaxation.

I sent Codex the follow-up. It came back with a fix that worked. The modals opened. The dev server was clean. I almost committed it.

Then I read the diff.

## The dirty fix

Codex had moved the Trusted Types policy to an external file, which was correct. It had also, without being asked, switched the entire CSP delivery from an HTTP header in the `_headers` file to a `<meta http-equiv>` tag generated by SvelteKit's `kit.csp` config. The reason it did this is that SvelteKit can compute SHA-256 hashes of its own internal hydration scripts at build time and bake them into the meta tag, which makes the strict CSP work without needing nonces or an external policy.

The problem is that meta tag CSP is weaker than header CSP in three specific ways. The `frame-ancestors` directive is silently ignored when CSP is delivered as a meta tag, which means the clickjacking protection I had carefully written into the policy was now a no-op. The `report-uri` and `report-to` directives are also ignored, which means the live CSP violation dashboard I am planning to build for the attack mode demo cannot use real browser-emitted reports. And meta tag CSP only takes effect once the HTML parser reaches the tag, so anything injected before that runs unrestricted, while header CSP applies from the very first byte of the response.

A judge inspecting the response headers would see CSP in a meta tag and immediately understand what that means. Credibility gone, on the exact dimension I am trying to win on.

So I told Codex to undo it. Not by reverting blindly, but by investigating whether the static adapter SvelteKit was using could produce a build that satisfied all three constraints I cared about: header-delivered CSP, no meta tag, and zero inline scripts. I asked it to stop and tell me if it hit a wall.

It hit a wall. Honestly. It came back and said: "with the current static adapter, I cannot produce a build that is both fully functional and meets your header-only CSP requirement without the meta-tag tradeoff." It offered three architectural options. Switch to a server adapter. Accept the meta tag. Redesign the app to avoid hydration entirely.

This is the thing I want to call out about working with AI coding tools. The right answer was to have it stop at the wall and surface the decision, not to have it paper over the constraint. The honest "I cannot do this" response is more valuable than ten clever workarounds.

I picked option one. Switch from `@sveltejs/adapter-static` to `@sveltejs/adapter-cloudflare`. This means every request to ShieldCV runs through a thin Cloudflare Worker that generates a fresh random nonce per request, sets the CSP header on the response with that nonce inside `script-src` alongside `strict-dynamic`, and serves the same static HTML that the static adapter would have served. The other security headers get set by a SvelteKit `hooks.server.ts` handler that runs on every response. The worker never touches user data. ShieldCV is still local-first because all resume processing happens in the browser, in encrypted IndexedDB, after the page has finished loading. The worker exists purely to harden the delivery layer.

Codex made the switch in one pass. Wrangler config, hooks file, svelte.config.js updated with `kit.csp: { mode: 'nonce' }`, Cloudflare adapter installed, the deleted `_headers` file replaced with a smaller one that handles only static asset responses. The build came back clean. The dev server came up.

## The proof

I opened the browser, clicked each of the four security posture cards, and each modal opened with the right copy and closed correctly with the X button, with Escape, and with a click outside. Then I opened the DevTools console and typed this:

```javascript
document.body.innerHTML = '<img src=x onerror=alert(1)>'
```

This is the standard XSS payload. On almost any web app on the internet, this will pop an alert box. On ShieldCV, the browser threw a TypeError I had written into the Trusted Types default policy:

```
Uncaught TypeError: ShieldCV blocked unsafe Trusted Types HTML assignment.
  at rejectAssignment (trusted-types.js:7:11)
  at createHTML (trusted-types.js:12:25)
```

The XSS payload was rejected at the API boundary. The alert never fired. The image element was never created. The string never made it to the DOM.

Then I exported a HAR of the page load and counted the requests. Seventy four requests, every single one to localhost. Zero external. No analytics, no fonts from a CDN, no third-party scripts, no telemetry, no error tracking. The local-first promise made literal and verifiable from the network trace.

The response headers on the main document showed every security header I had asked for. Header-delivered CSP with a fresh nonce. Strict-Transport-Security with preload. X-Content-Type-Options. X-Frame-Options. Referrer-Policy. Permissions-Policy denying every dangerous browser API. Cross-Origin-Opener-Policy, Cross-Origin-Embedder-Policy, Cross-Origin-Resource-Policy all set to the strictest values that still let the app function. This is a more complete defensive header stack than most production web apps ever ship.

Pushed Prompt 1. CI passed. Security workflow passed. End of day one.

## What is actually here

Two prompts done out of eleven. Roughly twenty days left in the build. The visible product right now is small: a home page, four interactive cards explaining what is and is not being enforced, four placeholder routes for the features that come next. But the foundation is real. The CSP is real. The Trusted Types policy is real and I have proven it in the console with a payload that would have worked anywhere else. The cross-origin isolation headers are real. The dependency tree is OSV-clean and pinned to a coherent set of major versions.

The next prompt is the crypto package. WebCrypto wrappers, AES-GCM with authenticated encryption, PBKDF2 at six hundred thousand iterations to derive a key from a user passphrase, a versioned bundle format so the algorithm can be migrated later, constant-time hex comparison, and one hundred percent test coverage. Zero runtime dependencies. After that comes the encrypted IndexedDB storage layer. After that, the resume builder, the AI scanning, the HIPAA PHI detection, the GDPR rights tracker, the hash-chained audit log, the attack mode demo, and the deployment pipeline with a signed SBOM.

I am writing this series partly because I think the build is going to be useful to other people who are trying to ship security-critical software with AI tools. The real work is not in making the AI write code. The real work is in catching the moments where it tries to silently weaken something you said you cared about. Every prompt today had at least one of those moments. The CSP downgrade was the biggest. There will be more. I will write about each of them.

## A note on Waypoint

ShieldCV is a side project. It is also a deliberate demonstration of what Waypoint Compliance Advisory exists to enable. The HIPAA scanner that catches PHI in clinical descriptions is the same kind of work I do for healthcare clients in my day job. The CMMC awareness module is the same kind of work I am pursuing CCA credentials for. The local-first architecture is the kind of thing I would tell a client to demand from any vendor handling sensitive data. The compliance artifacts that will ship with the repo, the DPIA, the threat model, the signed SBOM, the data flow diagram, are the artifacts I generate for clients during gap assessments.

If you found this post because you are responsible for compliance at a small or mid-size organization and you are wondering whether the security thinking I am applying to a hackathon project translates to the work I would do for you, the answer is yes. [Waypoint](https://waypointca.com) is open for engagements.

But this series is mostly about the build. The next post will probably be about the crypto package, because that is where the security depth starts to show.

The repo is at [github.com/WaypointCA/shieldcv](https://github.com/WaypointCA/shieldcv). It is public. Watch it if you want to see this happen in real time.

DPE. GSD. One prompt at a time.
