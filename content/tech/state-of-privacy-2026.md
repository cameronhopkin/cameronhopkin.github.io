---
title: "The State of Privacy in 2026: How We Got Here"
date: 2026-01-21
description: "Part 1: The $300 billion surveillance economy and why US law isn't protecting you."
tags: ["privacy", "cybersecurity", "data-brokers"]
draft: false
---

# The State of Privacy in 2026: How We Got Here

January 21, 2026 · 8 min read

#Privacy #Cybersecurity #DataBrokers

*Part 1 of 2 — [Part 2: Tools That Actually Work](/practical-privacy-tools-2026)*

I've been in cybersecurity for over two decades. I've watched the industry evolve from "don't click suspicious links" to a $300 billion surveillance economy that most people don't even know exists.

This isn't about paranoia. It's about understanding what's actually happening with your data, why the law isn't protecting you, and what that means for your daily life.

## The $300 Billion Industry You've Never Heard Of

There's an entire industry built around buying, selling, and trading your personal information. They're called data brokers, and according to [Grand View Research](https://www.grandviewresearch.com/industry-analysis/data-broker-market-report), the market hit $278 billion in 2024 and is projected to reach $512 billion by 2033.

Let that sink in. Half a trillion dollars—built on your data.

Companies like Acxiom, Experian, Oracle, and thousands of smaller players collect information from every digital touchpoint in your life. They aggregate it, analyze it, and sell it to anyone willing to pay. Oracle alone claims to have data on over [two billion people globally](https://policyreview.info/articles/analysis/untamed-and-discreet-role-data-brokers-surveillance-capitalism-transnational-and), with the ability to infer more than 30,000 attributes about each person.

Where does your data come from? Everywhere:
- Public records (property, court, voter registration)
- Social media activity
- Purchase histories
- Mobile apps and location data
- Website cookies and tracking pixels
- Loyalty programs and store cards
- Financial transactions
- Health and fitness apps

The result? Detailed dossiers on hundreds of millions of Americans, sold to marketers, insurance companies, employers, landlords, law enforcement, and anyone else with a credit card.

And here's the uncomfortable part: it's almost entirely legal.

## The Legal Vacuum

The United States has no comprehensive federal privacy law.

Let me say that again: in 2026, the world's largest tech economy has no baseline federal protection for your personal data.

The [American Data Privacy and Protection Act (ADPPA)](https://en.wikipedia.org/wiki/American_Data_Privacy_and_Protection_Act) came closest in 2022—it passed committee with a 53-2 vote, the first federal privacy bill ever to do so. Then it died. The [American Privacy Rights Act (APRA)](https://www.techtarget.com/searchsecurity/tip/State-of-data-privacy-laws) followed in 2024. Same result.

Why? Politics. Disagreements over whether federal law should override stronger state laws. Disputes about whether individuals should be able to sue companies directly. Lobbying from industries that profit from the status quo.

The result is a patchwork. As of January 2026, [19 states have comprehensive privacy laws](https://insights.manageengine.com/privacy-compliance/us-state-data-privacy-laws/)—California, Colorado, Connecticut, Delaware, Indiana, Iowa, Kentucky, Maryland, Minnesota, Montana, Nebraska, New Hampshire, New Jersey, Oregon, Rhode Island, Tennessee, Texas, Utah, and Virginia.

Each law is slightly different. Each has different definitions, different exemptions, different enforcement mechanisms. If you're a business, compliance is a nightmare. If you're a consumer, protection depends entirely on where you live.

Here's what's happening right now:

**California's Delete Act** launched in January 2026, creating a [one-stop deletion portal](https://iapp.org/news/a/new-year-new-rules-us-state-privacy-requirements-coming-online-as-2026-begins) where residents can request removal from all registered data brokers at once. It's a good idea—and it only applies to Californians.

**Montana closed the law enforcement loophole** in 2025, becoming the first state to [prevent police from buying citizens' data](https://insights.manageengine.com/privacy-compliance/us-state-data-privacy-laws/) from brokers when they'd normally need a warrant. In every other state? Law enforcement can simply purchase what they can't legally collect.

**The Global Privacy Control (GPC)** is now effectively mandatory in California, Colorado, Connecticut, and Oregon. Companies that ignore browser opt-out signals face [seven-figure settlements](https://www.ketch.com/blog/posts/us-privacy-laws-2026). Healthline Media paid over $1.5 million in 2025 for failing to honor GPC signals.

**Children's data** is getting more attention, with several states now requiring opt-in consent for collecting teen data or banning targeted advertising to minors entirely.

But here's the fundamental problem: these laws focus on giving you the right to *opt out*. They don't stop the collection in the first place. The default is surveillance. Privacy requires effort.

## Your Email Is Your Identity

Most people think privacy is about hiding their IP address. It's not.

Your email address is your primary online identity. Not your IP. Not your browser fingerprint. Your email.

When you use the same Gmail address for your bank, social media, shopping, newsletters, and random website signups, you've created a single thread that ties your entire digital life together. Data brokers don't need sophisticated tracking. They just need your email, and they can cross-reference everything.

That email shows up in breach databases. It gets sold and resold. It connects your purchase history to your location data to your browsing behavior to your social media activity. That's how "personalized" ads feel like surveillance—because they are.

Check [Have I Been Pwned](https://haveibeenpwned.com/) right now. Enter your primary email. Count the breaches. Now understand that each of those breaches fed information into an ecosystem designed to build a profile of you.

## The VPN Myth

VPNs have been marketed as privacy silver bullets. They're not.

Here's what a VPN actually does:
- Encrypts traffic between your device and the VPN server
- Masks your IP from websites you visit
- Protects you on public Wi-Fi

Here's what a VPN does NOT do:
- Make you anonymous
- Stop tracking when you're logged into Google, Facebook, or any service
- Prevent data brokers from collecting your information
- Hide your activity from the VPN provider

That last point matters. When you use a VPN, you're shifting trust from your ISP to the VPN provider. You're still trusting someone. The question is whether that trade makes sense.

The "no-log" claims VPN companies make? [Technically impossible](https://symlexvpn.com/common-vpn-myths-debunked/) at scale. All VPNs keep some operational data to function. The better question is: what do they keep, for how long, and under what jurisdiction?

## Even "Private" Services Have Limits

I use and recommend Proton products. But I'm not going to pretend they're magic.

Proton cooperates with law enforcement when legally compelled by Swiss authorities. This has happened multiple times.

In 2021, Proton provided the [IP address and device details](https://www.cpomagazine.com/data-privacy/request-from-french-police-compelled-protonmail-to-reveal-ip-logs/) of a French climate activist to Swiss authorities after a request through Europol. The user was subsequently arrested.

In 2024, Proton disclosed a user's [recovery email address](https://www.techradar.com/computing/cyber-security/proton-mail-hands-data-police-again-is-it-still-safe-for-activists) to Spanish police investigating a Catalan independence activist. That recovery email led to identification and arrest.

According to [Proton's transparency report](https://proton.me/legal/transparency), in 2025 they received 9,301 legal orders for ProtonMail data and complied with 8,313 of them.

What Proton *cannot* provide, even when compelled:
- Contents of encrypted emails (they literally can't decrypt them)
- VPN activity logs (ProtonVPN denied all 59 requests in 2025)

What they *can* be compelled to provide:
- Recovery email addresses
- IP addresses (if logging is enabled)
- Account creation dates
- Email metadata (sender, recipient, subject lines, timestamps)

This isn't a criticism of Proton—they're more transparent than almost any provider, and their encryption is real. But "encrypted" and "anonymous" are different things. If you're a regular person trying to escape advertising surveillance, Proton is a massive upgrade from Gmail. If you're an activist with government-level adversaries, you need to understand these limits.

## Why It Matters

You might be thinking: "I have nothing to hide."

That's not the point. Privacy isn't about hiding wrongdoing. It's about:

**Control.** Who decides what happens with information about you? Right now, the answer is: thousands of companies you've never heard of.

**Context.** Information appropriate in one context can be harmful in another. Your health data might be fine with your doctor but problematic with your employer or insurance company.

**Power asymmetry.** Companies and governments know everything about you. You know almost nothing about what they know or how they use it.

**Future uncertainty.** Data collected today can be used in ways you can't predict. What's harmless now might not be harmless under different circumstances, different laws, or different leadership.

The $300 billion data broker industry exists because your information has value. Every piece of data you give away—or that gets collected without your knowledge—feeds a system designed to predict and influence your behavior.

You can't opt out of the digital world. But you can be intentional about what you share and with whom.

## What Now?

The legal landscape isn't going to save you anytime soon. Federal privacy legislation remains gridlocked. State laws help if you're lucky enough to live in the right state, but even then, they're reactive—requiring you to actively opt out rather than protecting you by default.

The practical reality is this: if you want privacy in 2026, you have to build it yourself.

That means making different choices about the tools you use, the services you trust, and the defaults you accept. It requires discipline, precision, and execution.

In Part 2, I'll cover the specific tools and tactics that actually work—email aliases, DNS-level blocking, browser hardening, password management, and more. Not paranoid overkill, but practical steps that meaningfully reduce your exposure without requiring a computer science degree.

Your privacy is worth the effort.

---

**Continue to [Part 2: Practical Privacy Tools That Actually Work](/practical-privacy-tools-2026)**

---

## Sources

- [Grand View Research: Data Broker Market Report](https://www.grandviewresearch.com/industry-analysis/data-broker-market-report)
- [IAPP: US State Privacy Laws Overview](https://iapp.org/resources/article/us-state-privacy-laws-overview)
- [IAPP: New Year, New Rules for 2026](https://iapp.org/news/a/new-year-new-rules-us-state-privacy-requirements-coming-online-as-2026-begins)
- [Ketch: Data Privacy Laws 2026](https://www.ketch.com/blog/posts/us-privacy-laws-2026)
- [TechTarget: US Data Privacy Protection Laws Guide](https://www.techtarget.com/searchsecurity/tip/State-of-data-privacy-laws)
- [ManageEngine: US State Privacy Laws 2025-2026](https://insights.manageengine.com/privacy-compliance/us-state-data-privacy-laws/)
- [Proton Transparency Report](https://proton.me/legal/transparency)
- [CPO Magazine: ProtonMail IP Logs Case](https://www.cpomagazine.com/data-privacy/request-from-french-police-compelled-protonmail-to-reveal-ip-logs/)
- [TechRadar: Proton Mail Catalan Activist Case](https://www.techradar.com/computing/cyber-security/proton-mail-hands-data-police-again-is-it-still-safe-for-activists)

---

*Questions? Find me on [LinkedIn](https://linkedin.com/in/cameronhopkin).*
