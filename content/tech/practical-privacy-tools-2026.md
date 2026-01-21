---
title: "Practical Privacy Tools That Actually Work in 2026"
date: 2026-01-21
description: "Part 2: Email aliases, DNS blocking, and the tools that actually reduce your exposure."
tags: ["privacy", "cybersecurity", "tools"]
draft: false
---

# Practical Privacy Tools That Actually Work in 2026

January 21, 2026 · 9 min read

#Privacy #Cybersecurity #Tools

*Part 2 of 2 — [Part 1: The State of Privacy](/state-of-privacy-2026)*

In Part 1, I covered the problem: a $300 billion data broker industry, no federal privacy law, and a digital ecosystem designed to track everything you do.

Now let's talk solutions. Not paranoid overkill—practical tools and habits that meaningfully reduce your exposure without requiring you to live off the grid.

## Email Aliases: The Highest-Impact Change

If you do only one thing from this article, make it this.

Email aliases let you generate unique email addresses that forward to your real inbox. Instead of giving a website your actual email, you give them `shopping-xyz123@simplelogin.io`. Emails arrive in your inbox. You can reply from the alias. And if that alias gets compromised, sold, or starts receiving spam, you disable it.

**Why this matters:**

Your email is your primary online identifier. Data brokers cross-reference your email across databases to build profiles. One email address = one thread connecting everything about you.

With aliases:
- **Breach containment.** When a service gets hacked, attackers get a disposable address. Your core email stays safe.
- **Spam source identification.** Use unique aliases per service. When spam arrives, you know exactly who sold your data.
- **No cross-site tracking.** Each service sees a different email. Brokers can't connect the dots.

**Tools:**

[SimpleLogin](https://simplelogin.io/) — My primary recommendation. Now owned by Proton. Free tier gives you 10-15 aliases. Premium ($30/year) is unlimited with custom domains, PGP encryption, and more. Open source, independently audited.

[Addy.io](https://addy.io/) — Solid open source alternative. Generous free tier.

[Firefox Relay](https://relay.firefox.com/) — Mozilla's offering. Simpler but more limited.

**The plus-sign trick doesn't cut it.** Using `yourname+shopping@gmail.com` is better than nothing, but trivial to strip. Your real email is exposed. True aliases provide actual separation.

**My setup:** I organize aliases by category—`news@` for newsletters, `security@` for industry subscriptions, `shopping@` for retail, `finance@` for money-related accounts. Each routes to a folder. Intentional consumption, not reactive inbox chaos.

## Get Off Free Email

Gmail's business model is advertising. Google scans your emails to build a profile that advertisers pay to target. They're not selling your emails—they're using your emails to sell access to your attention.

The same applies to Outlook.com, Yahoo, and other free providers. If you're not paying, you're the product.

**Paid alternatives:**

[ProtonMail](https://proton.me/mail) — Swiss-based, end-to-end encrypted, can't read your email contents even if compelled. Free tier available, paid plans start around $4/month. I covered their law enforcement limitations in Part 1—understand them, but for most people this is a massive upgrade from Gmail.

[Tutanota](https://tutanota.com/) — German-based alternative. Similar encryption model. Slightly cheaper.

[Fastmail](https://www.fastmail.com/) — Australian-based. Not end-to-end encrypted, but no advertising model. Strong on features and reliability.

**Migration approach:**
1. Create your new account and get comfortable with it
2. Set up forwarding from your old email
3. Update critical accounts first (banking, healthcare, tax services)
4. Gradually migrate everything else
5. Let the old account become a legacy catch-all

This isn't a weekend project—it takes weeks to do properly. But the result is removing one of the biggest privacy compromises most people make daily.

## DNS-Level Blocking

Every website you visit starts with a DNS query—your device asking "what's the IP address for this domain?" By default, your ISP sees every one of these queries.

Switching to a privacy-focused DNS with built-in blocking stops trackers and ads at the network level, before they ever reach your browser. This works across all apps, not just your browser.

**Free options:**

[NextDNS](https://nextdns.io/) — 300,000 queries/month free (plenty for most users). Blocks ads, trackers, malware. Customizable blocklists. Takes about 5 minutes to set up. Works on all devices.

[AdGuard DNS](https://adguard-dns.io/) — Completely free, no account required. Three modes: default (ads/trackers), family (adds adult content filtering), and non-filtering (just encrypted DNS).

[Quad9](https://quad9.net/) — Free, nonprofit. Blocks malicious domains. Based in Switzerland. Strong privacy policy but no ad blocking.

**Setup:** You can configure DNS per-device, or set it at your router level for whole-home coverage. Most services provide setup guides for every platform.

DNS blocking won't catch everything—sophisticated trackers can bypass it—but it stops a huge amount of garbage before it touches your device.

## Browser Hardening

Your browser is ground zero for tracking. A few changes make a significant difference.

**Browser choice:**

[Firefox](https://www.mozilla.org/) remains the best balance of usability and privacy. It's the only major browser not built on Google's Chromium codebase, which matters as Google continues tightening control over what extensions can do.

Configure Firefox with [recommended privacy settings](https://www.privacytools.io/private-browser):
- Enhanced Tracking Protection set to Strict
- Send "Do Not Track" signals
- Delete cookies on close (if you can tolerate re-logging into sites)
- Disable telemetry

[Tor Browser](https://www.torproject.org/) — For when you need actual anonymity, not just privacy. Significantly slower but routes traffic through multiple encrypted relays. Use it for specific tasks, not daily browsing.

**Extensions:**

[uBlock Origin](https://ublockorigin.com/) — The gold standard for ad and tracker blocking. Open source, lightweight, highly configurable. [Scored 100/100](https://cyberinsider.com/best-ad-blocker/) in recent blocking tests. Note: Google's Manifest V3 changes have degraded extension capabilities in Chrome-based browsers. Firefox still supports the full-featured version.

[Privacy Badger](https://privacybadger.org/) — Made by the Electronic Frontier Foundation. Learns to block trackers based on behavior rather than static lists. Good complement to uBlock Origin.

[LocalCDN](https://www.localcdn.org/) or [Decentraleyes](https://decentraleyes.org/) — Prevents tracking through content delivery networks by serving common libraries locally.

**Keep extensions minimal.** Every extension you add makes your browser fingerprint more unique. Install what you need, nothing more.

## Password Management

If you're reusing passwords or storing them in your browser, stop. A password manager is non-negotiable for basic security hygiene.

**Free:**

[Bitwarden](https://bitwarden.com/) — Open source, cross-platform, genuinely useful free tier. Premium is $10/year if you want extras like built-in 2FA codes and encrypted file storage.

[KeePassXC](https://keepassxc.org/) — Fully offline, open source. No cloud, no account, no sync. You manage the database file yourself. Maximum control, but more manual.

**Paid:**

[1Password](https://1password.com/) — $36/year for individuals, $60/year for families. Polished interface, excellent browser integration, strong security model. If you'll actually use a password manager because it's pleasant to use, 1Password is worth the money.

[Proton Pass](https://proton.me/pass) — Included with Proton subscriptions. Integrates well with their ecosystem. Built-in email alias generation.

**The point isn't which manager you choose—it's that you use one.** Generate unique, random passwords for every account. Let the manager remember them. Your brain is not a secure storage medium.

## Two-Factor Authentication

SMS-based 2FA is better than nothing, but [SIM-swapping attacks are real](https://krebsonsecurity.com/category/sim-swapping/) and common. Use an authenticator app instead.

**Free:**

[Aegis Authenticator](https://getaegis.app/) (Android) — Open source, encrypted backups, better than Google Authenticator in every way.

[2FAS](https://2fas.com/) (iOS/Android) — Open source, clean interface, cloud backup option.

**Paid/Integrated:**

[1Password](https://1password.com/) — Can store TOTP codes alongside passwords. Convenient, though security purists argue you shouldn't keep eggs in one basket.

[Proton Pass](https://proton.me/pass) — Same deal. Integrated 2FA with your Proton account.

**Enable 2FA on everything important:** email, banking, social media, cloud storage. Prioritize authenticator apps over SMS.

## Secure Messaging

Standard SMS is unencrypted and trivially intercepted. For sensitive conversations:

[Signal](https://signal.org/) — End-to-end encrypted messaging. Free, open source, minimal metadata collection. [Proven in court](https://signal.org/bigbrother/) that they have almost nothing to provide when subpoenaed (just phone number, account creation date, and last connection time).

This isn't about having something to hide. It's about having conversations that aren't stored in plaintext on carrier servers indefinitely.

## Data Broker Opt-Outs

You can request removal from data brokers, though it's tedious.

**DIY:**

[Have I Been Pwned](https://haveibeenpwned.com/) — Check which breaches contain your email. Also offers notification when your email appears in new breaches.

[Privacy Rights Clearinghouse](https://privacyrights.org/data-brokers) — Maintains a list of data brokers with opt-out procedures. Manual but free.

**California residents:** The [Delete Act portal](https://iapp.org/news/a/new-year-new-rules-us-state-privacy-requirements-coming-online-as-2026-begins) (launching 2026) lets you submit one request to all registered brokers.

**Paid services:**

[DeleteMe](https://joindeleteme.com/) — ~$129/year. They handle opt-out requests on your behalf and provide regular reports.

[Privacy Duck](https://www.privacyduck.com/) — Similar service, slightly different broker coverage.

These services don't eliminate your data from existence, but they significantly reduce your exposure in people-search sites and marketing databases.

## The Four-Week Roadmap

Theory without execution is useless. Here's a practical sequence:

### Week 1: Audit
- Check [Have I Been Pwned](https://haveibeenpwned.com/) for your primary email
- List every service using that email (export your password manager, search inbox for "welcome" and "confirm")
- Identify high-risk accounts: banking, healthcare, tax, primary email, cloud storage

### Week 2: Foundation
- Install a password manager if you don't have one
- Enable 2FA on your most important accounts (email first, then banking, then everything else)
- Generate new unique passwords for high-risk accounts

### Week 3: Email Aliases
- Create a [SimpleLogin](https://simplelogin.io/) or [Addy.io](https://addy.io/) account
- Install the browser extension
- Start using aliases for all new signups immediately
- Begin migrating existing accounts (one category per week)

### Week 4: DNS and Browser
- Switch to [NextDNS](https://nextdns.io/) or [AdGuard DNS](https://adguard-dns.io/)
- Install [uBlock Origin](https://ublockorigin.com/)
- Review browser privacy settings

### Ongoing: Email Migration
- Research privacy-focused email providers
- Create new account, set up forwarding
- Update critical accounts first
- Let old account become legacy catch-all

## The Bottom Line

Privacy in 2026 isn't about achieving invisibility. It's about making intentional choices about what you share and with whom.

These tools aren't magic. They're layers. Each one reduces your exposure. Combined, they meaningfully change your relationship with the surveillance economy.

None of this requires technical expertise. It requires discipline, precision, and execution.

Start this week. Pick one thing. Do it. Then do the next thing.

Progress, not perfection.

---

## Quick Reference

| Category | Free | Paid |
|----------|------|------|
| **Email Aliases** | [SimpleLogin](https://simplelogin.io/), [Addy.io](https://addy.io/) | SimpleLogin Premium ($30/yr) |
| **Private Email** | [ProtonMail](https://proton.me/mail) free tier | ProtonMail ($48/yr), [Fastmail](https://www.fastmail.com/) ($50/yr) |
| **DNS Blocking** | [NextDNS](https://nextdns.io/), [AdGuard DNS](https://adguard-dns.io/) | NextDNS Pro ($20/yr) |
| **Password Manager** | [Bitwarden](https://bitwarden.com/), [KeePassXC](https://keepassxc.org/) | [1Password](https://1password.com/) ($36/yr) |
| **2FA** | [Aegis](https://getaegis.app/), [2FAS](https://2fas.com/) | Built into 1Password/Proton Pass |
| **Browser** | [Firefox](https://www.mozilla.org/) | — |
| **Extensions** | [uBlock Origin](https://ublockorigin.com/), [Privacy Badger](https://privacybadger.org/) | — |
| **Messaging** | [Signal](https://signal.org/) | — |
| **Breach Check** | [Have I Been Pwned](https://haveibeenpwned.com/) | — |
| **Data Removal** | DIY via [Privacy Rights](https://privacyrights.org/data-brokers) | [DeleteMe](https://joindeleteme.com/) ($129/yr) |

---

*Questions? Find me on [LinkedIn](https://linkedin.com/in/cameronhopkin).*
