---
layout: post
title: "API & Web Security: Notes from the OWASP API Top 10 Trenches"
date: 2026-06-15
categories: [API Security, Web Security]
tags: [api-security, owasp, oauth, bola, ssrf, mass-assignment, certification]
excerpt: "A modular study guide to API and web application security — OWASP API Top 10, OAuth, business-logic flaws, and the certification tracks that prove you can break them."
---

<style>
  .kw  { color: #ffd700; font-weight: bold; }
  .red { color: #ff4444; font-weight: bold; }
  .grn { color: #00ff88; font-weight: bold; }
  .prp { color: #7b2fff; font-weight: bold; }
  .cyan{ color: #00d4ff; font-weight: bold; }
</style>

# <span class="cyan">API</span> & Web Security, Notes from the Trenches

APIs are where the modern attack surface actually lives. The slick single-page frontend is just a painted door, the real logic, the real data, and the real vulnerabilities sit behind the JSON endpoints feeding it. Break the API and you usually do not need the frontend at all.

This post introduces my study guide for API and web application security:

> **Repository:** [github.com/botesjuan/API-Web-Security](https://github.com/botesjuan/API-Web-Security)

It is a structured, modular set of notes that bridges the theory, the OWASP frameworks and OAuth flows, with the hands-on testing techniques you use to exploit them. It is aligned to several professional certification tracks, so it doubles as exam prep.

## The OWASP API Security Top 10

The spine of the repo is the OWASP API Security Top 10, the categories that account for the overwhelming majority of real-world API breaches:

- **Broken Object Level Authorization (<span class="red">BOLA</span>)** — the number one API killer. Change the `id` in the request and read someone else's data. Simple, devastating, everywhere.
- **Broken Authentication** — weak tokens, missing expiry, guessable credentials, and broken session handling.
- **Broken Object Property Level Authorization** — including <span class="red">mass assignment</span>, where you set fields the API never meant to expose, like `"isAdmin": true`.
- **Unrestricted Resource Consumption** — missing rate limits that open the door to brute force and denial of wallet.
- **Broken Function Level Authorization** — calling the admin endpoint as a normal user.
- **Server-Side Request Forgery (<span class="red">SSRF</span>)** — making the API fetch internal resources and cloud metadata on your behalf.
- **Security Misconfiguration**, **Improper Inventory Management** (shadow and zombie APIs), and **Unsafe Consumption of APIs**.

## Beyond the Top 10

The notes go further than the standard list to cover the patterns that show up in real engagements:

- **OAuth 2.0 and authentication flows** — the implicit grant pitfalls, redirect URI abuse, and token leakage that plague real implementations.
- **Business logic flaws** — the vulnerabilities no scanner will ever find, because the request is perfectly valid and the *logic* is what is broken.
- **Race conditions** — exploiting the tiny window between check and use to redeem a coupon twice or overdraw a balance.
- **Web cache deception** and **NoSQL / MongoDB injection**.
- **LLM API exploitation** — prompt injection and abuse as AI endpoints get bolted onto everything.

## Certification tracks

The material maps onto multiple API and web security certifications, so the same notes serve double duty as exam prep:

- <span class="prp">CASA</span> — Certified API Security Analyst
- <span class="prp">ASCP</span> — API Security Certified Professional
- <span class="prp">ACP</span> — APIsec Certified Practitioner
- <span class="prp">BSCP</span> — Burp Suite Certified Practitioner

It leans heavily on established, free resources like the PortSwigger Web Security Academy and the OWASP project materials, structured into a path you can actually follow.

## How to test an API in practice

1. **Find the documentation.** Swagger, OpenAPI, Postman collections, even an old `/docs` endpoint. The spec is the map of the attack surface.
2. **Enumerate every endpoint and method.** Hidden verbs and undocumented routes are where the soft spots hide.
3. **Attack authorization first.** <span class="red">BOLA</span> and function-level authorization flaws are the highest-yield bugs, test every object reference against a second account.
4. **Probe the logic.** Replay, reorder, and race the requests. Tamper with quantities, prices, and state transitions.
5. **Prove impact, not just presence.** A finding without demonstrated impact is just a theory. Read the other user's data, escalate the privilege, exfiltrate the secret.

## Why it matters

Whether you are preparing for an API security certification, hardening your own organisation's endpoints, or running a live engagement, the failure modes are the same handful repeating endlessly. Learn them once, deeply, and you will spot them in every product you touch.

> **Get it here:** [API-Web-Security](https://github.com/botesjuan/API-Web-Security)
