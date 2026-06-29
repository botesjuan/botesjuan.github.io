---
layout: post
title: "Being a Unicorn with CISSP as a Penetration Tester"
date: 2024-10-0
categories: [Certification, GRC]
tags: [cissp, certification, risk-management, security-governance, study-notes]
excerpt: "How a hands-on penetration tester thinks like a manager long enough to pass the CISSP, the eight security domains, the art of the distractor, and a public set of study notes with the of AI LLM study partner prompts."
---

<style>
  .kw  { color: #ffd700; font-weight: bold; }
  .red { color: #ff4444; font-weight: bold; }
  .grn { color: #00ff88; font-weight: bold; }
  .prp { color: #7b2fff; font-weight: bold; }
  .cyan{ color: #00d4ff; font-weight: bold; }
</style>

# Passing the <span class="cyan">CISSP</span> as a Penetration Tester

The CISSP is a strange exam for an offensive operator. My instinct on every question is "how do I exploit this," and the exam wants me to answer "how do I govern this." Passing it in 2024, after eight months of part-time study, meant learning to switch off the attacker brain and think like a risk manager wearing a suit.

This post introduces my public study notes for anyone walking the same road:

> **Repository:** [github.com/botesjuan/CISSP-Studies](https://github.com/botesjuan/CISSP-Studies)

## Think like a manager, not a hacker

The single most important mindset shift, the CISSP is not a technical exam, it is a *management* exam. ISC2 wants the answer a senior security leader would give. That means:

- **People before technology.** The "best" answer is frequently policy, training, or management approval, not the firewall rule.
- **Risk before fixes.** You assess and quantify risk before you spend a cent on a control.
- **Root cause before symptom.** The exam loves to dangle a satisfying technical fix that treats the symptom while a process answer treats the cause.

For a penetration tester this is genuinely uncomfortable, your gut screams "patch it," and the exam rewards "evaluate the business impact first."

## The eight domains

The notes cover all eight CISSP Common Body of Knowledge domains, with a visual whiteboard diagram for each:

1. **Security and Risk Management**
2. **Asset Security**
3. **Security Architecture and Engineering**
4. **Communication and Network Security**
5. **Identity and Access Management (IAM)**
6. **Security Assessment and Testing**
7. **Security Operations**
8. **Software Development Security**

Domains 1 and 3 carry the most weight and the most pain. Risk Management is the philosophical core of the whole exam, get its mindset right and the other seven domains fall into place.

## The art of the distractor

The hardest part of the CISSP is not knowledge, it is the question phrasing. ISC2 are masters of the *distractor*, an answer that is technically correct but not the **best** answer. The repo includes an analysis of the keyword phrasing the exam leans on, words like:

> <span class="kw">best</span> · <span class="kw">most</span> · <span class="kw">primarily</span> · <span class="kw">first</span> · <span class="red">not</span> · <span class="kw">least</span> · <span class="kw">except</span>

Spotting these words and re-reading the question through the "manager's lens" is what separates a pass from a near miss. Often two answers are correct and you must pick the one that is correct *first* or *best*.

## How I studied

The repo documents the exact method that worked for me:

- Watch the domain video series end to end, then again at speed.
- Work through the official practice questions, domain by domain.
- Use an LLM as a study partner to generate scenario questions and, more usefully, to explain *why* the wrong answers are wrong.
- Build memory aids and mnemonics for the list-heavy material.
- Repeat the questions until the distractors stop fooling you.

Inside the repo you will find PDF study notes, the per-domain whiteboard diagrams, a curated video playlist, and sample exam-style questions with full explanations on topics like intangible assets, immutable infrastructure, service accounts, and OAuth 2.0.

## Why a hacker should bother

The CISSP will not teach you to exploit anything. What it does is force you to see the whole organisation, the policy, the people, the risk register, the budget, that surrounds the systems you break. That perspective makes you a sharper tester, because you finally understand the constraints the defenders are actually working under, and you write better reports because you can speak their language.

> **Get it here:** [CISSP-Studies](https://github.com/botesjuan/CISSP-Studies)
