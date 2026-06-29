---
layout: post
title: "BSCP Exam Study Field Guide for the Burp Suite Certified Practitioner"
date: 2023-03-25
categories: [Web Security, Certification]
tags: [bscp, burp-suite, xss, web-security, portswigger, certification]
excerpt: "My study notes as companion for the Burp Suite Certified Practitioner exam, with payloads, scripts, wordlists, and the practical tradecraft that gets you through the two-hour clock and achieving 100%."
---

<style>
  .kw  { color: #ffd700; font-weight: bold; }
  .red { color: #ff4444; font-weight: bold; }
  .grn { color: #00ff88; font-weight: bold; }
  .prp { color: #7b2fff; font-weight: bold; }
  .cyan{ color: #00d4ff; font-weight: bold; }
</style>

# <span class="cyan">BSCP</span> Exam Study, a Field Guide

The <span class="prp">Burp Suite Certified Practitioner</span> (BSCP) is PortSwigger's hands-on web security certification. There is no multiple choice, no theory dump, just two vulnerable web applications and a clock. You have to find the flaw, chain it, and steal the administrator's data before time runs out. It rewards muscle memory over memorisation.

This post introduces my open-source study companion for that exam:

> **Repository:** [github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study)

The repo has grown into one of the most referenced BSCP study resources on GitHub, and it exists for one reason, when you are sitting in the exam with the clock running, you do not want to be inventing a <span class="red">XSS</span> payload from scratch. You want it ready, tested, and pasteable.

## What is inside

The repository is organised around the way the exam actually plays out, not around a textbook table of contents. The BSCP hands you **two separate vulnerable web applications**, and you must fully compromise *both* — there is no partial credit, you need a clean **100%** to pass, which means walking the same four-phase kill chain to completion on each app inside the time limit.

### 1. Enumeration and scanning focus points
Every chain starts with mapping the application. Crawl the site, catalogue every parameter, cookie, header, and endpoint, and run a targeted Burp scan over the in-scope hosts to surface the obvious primitives. On the exam you are looking for the *one* foothold bug per app, so the focus is recognising the vulnerability class fast — an unsanitised input, a reflected parameter, a leaky API route — rather than boiling the ocean. The repo's wordlists and notes point you straight at the high-yield areas the PortSwigger labs reuse.

### 2. Gaining a foothold as low-privilege user "Carlos"
The first objective on each app is to break in as the standard low-privilege account, conventionally named **Carlos**. This is almost always a **client-side or authentication exploit** — a stored or reflected <span class="red">XSS</span> that steals Carlos's session cookie to your exploit server, a CSRF chain, an OAuth or password-reset flaw, or business-logic abuse. The payloads folder is built for exactly this moment: ready-to-fire exploit-server pages and cookie-exfiltration snippets you can paste and adapt rather than write under pressure.

### 3. Privilege escalation to administrator inside the web app
Holding Carlos is not enough — you must climb from low-privilege user to the **administrator** of the same application. This is where you steal or forge a higher-privilege token, abuse a flaw that exposes admin credentials, or pivot a second vulnerability (SQL injection, IDOR, broken access control, JWT weaknesses) into an admin session. The goal is reaching the admin-only functionality that the final phase depends on.

### 4. Data exfiltration as the administrator
With admin access secured, the last phase is reading the **sensitive flag** — the secret the exam scores you on. This typically means abusing an admin-only feature to achieve **command injection, file read, SSRF, or deserialization**, then exfiltrating the target data. Only when you have submitted the secret from *both* applications does the exam mark you complete.

Everything in the repo maps onto those four phases:

- **Payloads** — ready-to-fire snippets for the vulnerability classes that show up under exam conditions: cross-site scripting, SQL injection, SSRF, XXE, prototype pollution, request smuggling, and the authentication chains that tie them together.
- **Python** — small purpose-built scripts to automate the parts of the exam that are too slow to do by hand, such as brute-forcing a stay-logged-in cookie, scripting a blind data exfiltration, or generating malicious payloads on the fly.
- **Wordlists** — curated lists tuned for the PortSwigger labs and the exam environment rather than generic dictionaries.
- **Code & Extras** — supporting exploit servers, HTML attack pages, and reference notes.

## The two-stage exam model

The BSCP is graded in two stages, and understanding the structure is half the battle.

**Stage 1 — gain access to a user account.** You start as an anonymous or low-privilege user. The goal is to find a foothold vulnerability and pivot it into another user's session. This is usually where <span class="red">XSS</span>, CSRF, authentication flaws, and business-logic abuse live.

**Stage 2 — reach administrator and exfiltrate the secret.** Once you hold a normal account, you escalate to the <span class="grn">administrator</span>, then use admin-only functionality to read the contents of a file or the secret of another user. SQL injection, SSRF, deserialization, and file-read primitives dominate here.

The exam draws its vulnerabilities from the same pool as the [Web Security Academy](https://portswigger.net/web-security) labs. If you have genuinely solved the labs, you have seen every primitive the exam can throw at you, the challenge is recognising them quickly under pressure.

## How to use the repo to prepare

1. **Grind the Academy labs first.** The repo is a force multiplier, not a substitute. Solve the apprentice and practitioner labs until the patterns are reflexive.
2. **Build your own payload notebook.** Fork the repo, then rewrite the payloads in your own words. The act of curating them is what makes them stick.
3. **Practice the delivery, not just the discovery.** Knowing a stored <span class="red">XSS</span> exists is worthless if you cannot reliably exfiltrate an admin cookie to your exploit server within the time limit. Rehearse the full chain end to end.
4. **Time-box your mocks.** Sit the practice exam under a real two-hour clock. The pressure changes everything.

## Why it matters

Web application testing is the backbone of most penetration testing engagements, and the BSCP is one of the few certifications that proves you can actually do the work rather than describe it. The payloads and scripts in this repository are the same ones I reach for on live engagements, refined down to what works.

If it saves you ten minutes in the exam, it has done its job. Clone it, break it, and make it your own.

> **Get it here:** [Burp-Suite-Certified-Practitioner-Exam-Study](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study)
