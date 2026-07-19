---
layout: post
title: "AI & LLM skill up by Passing the INE eAIS AI Systems Security Specialist Exam"
date: 2026-07-19
categories: [AI Security, Certification]
tags: [eais, ine, ai-security, llm, prompt-injection, owasp-llm-top-10, red-team, certification]
excerpt: "A overview to INE's eAIS certification, a practical, hands-on exam that drops you into live LLM applications across four business sectors and asks you to attack them, audit them, and validate their controls like a real AI auditor / security analyst. What the exam actually tests, the mindset shift, and how to prepare, without spoilers."
---

<style>
  .kw  { color: #ffd700; font-weight: bold; }
  .red { color: #ff4444; font-weight: bold; }
  .grn { color: #00ff88; font-weight: bold; }
  .prp { color: #7b2fff; font-weight: bold; }
  .cyan{ color: #00d4ff; font-weight: bold; }
</style>

# AI with the <span class="cyan">eAIS</span> Certification

Every penetration tester has watched large language models grow in production systems over the last two years, chatbots wired into databases, agents given tools, retrieval pipelines feeding untrusted documents straight into a model's context. The AI attack surface is new, the trust boundaries are fuzzy, and most engagement checklists is maturing. INE's **eAIS — AI Systems Security Specialist** is one of the early certifications that actually makes you *do the work* against this new surface practically.

I passed eAIS with an overall score of **<span class="grn">93%</span>**, and this post is the guide I hope can help others: what the exam test, the mindset it demands, and how to prepare — with **zero exam spoilers**. Everything here is about *classes* of problems and skills, not answers.

## What the eAIS actually is

The eAIS sits at the AI LLM end of INE's **AI Systems Security Specialist** learning path, seven courses, roughly 32 hours of videos, and a hands-on labs. The exam itself is a **practical lab**, not a multiple-guess theory dump dressed up as rigor. You are handed a Kali machine with network access to a set of live AI applications and their supporting services, and you work through a series of challenges that force you to *investigate and test the real systems* and report your finding with an answer.

The thing that makes it feel like a genuine security audit exam is the framing. You are dropped into **independent scenarios set at different organisations, in different business sectors, each with a different role**. One moment you are a security auditor signing off an internal assistant before go-live; the next you are an independent assessor validating a whistle-blower's complaint against a live platform; then a secure-code reviewer checking whether a team's hardening actually holds. The scenarios are self-contained and can be done in any order — there is no linear "level 1 to level 10," just a shared lab and a set of systems to take apart.

Crucially, the questions are not all "exploit this." Some ask you to **attack**, some ask you to **audit**, and some ask you to **validate a control** and decide whether a defence is sound or merely theatrical. That mix is the whole point of the certification.

## The mindset shift: attacker *and* defender *and* engineer

If you come from a pure offensive background — as I do — the biggest adjustment is that the eAIS refuses to let you stay in the attacker's chair. The single largest domain on the exam is **Secure AI Design & Controls (30%)**, bigger than exploitation. For every attack you learn, the exam expects you to know the matching control and to recognise when a control is real versus when it just *looks* real.

That last part is where a lot of the exam's texture lives. Over and over you meet controls that pattern-match on surface form — a guardrail that blocks a phrase, a filter that recognises one way of writing something, a redactor that catches the obvious format. A red teamer's instinct is exactly the right tool here: reformulate, re-encode, come at it from an angle the pattern never anticipated. Knowing *why* a control is bypassable, and being able to state the correct fix, is what separates a pass from a fail.

The recurring theme that ties every scenario together is **trust boundaries**. User input is untrusted — everyone knows that. The eAIS drills the harder lesson: **retrieved RAG documents, tool outputs, and messages passed between agents are *also* untrusted**, and the moment a system treats any of them as instructions, it is broken. Indirect prompt injection — a payload planted in data the model will later read — is the signature attack of this space, and you need to recognise it in a vector store, in a tool's JSON response, and in a document a user uploaded.

## The five domains, and what each really means

The exam is scored across five weighted domains. Here is what each one is actually testing, in practical terms:

**1. AI/LLM Foundations for Security — 15%**
Can you look at a live LLM application and name its parts — model endpoint, orchestrator, retrieval layer, tools, guardrails — and tell how they are wired together? Can you tell *RAG-based retrieval* apart from *fine-tuning* apart from *base-model* behaviour just by probing the system and reading its logs? You need to be comfortable inspecting a vector store's configuration, tracing where prompts and sensitive data get stored, and reasoning about how embeddings, context windows, and prompt composition change the security picture. This was my lowest domain (<span class="kw">86%</span>) — the architecture and data-flow reasoning is subtle, and worth studying harder than its 15% weight suggests, because everything else builds on it.

**2. AI Abuse & Exploitation — 25%**
The offensive core. Direct prompt injection to override system instructions; **indirect** injection via retrieved content or tool output; exfiltrating system prompts, secrets, and RAG sources through crafted inputs; recognising jailbreak and policy-bypass patterns; spotting tool misuse and over-permissioning in agentic workflows; and understanding resource-exhaustion / "denial-of-wallet" abuse. If you have a red-team background this is your comfort zone, but the *indirect* and *agent-chain* variants are where people who only practised chatbot jailbreaks come unstuck.

**3. Secure AI Design & Controls — 30%**
The heaviest domain, and the one that makes eAIS more than a jailbreak cert. Least-privilege tool access; parameter validation and fail-closed tool calls; human-in-the-loop gates for high-impact actions; prompt/context separation (system vs developer vs user boundaries); structured-output schemas and validation; hardening RAG with source allowlisting, retrieval filtering, and provenance/citation; secrets handling (redaction, secure storage, never in prompts or logs); safe logging; and abuse-prevention controls like rate limiting and quotas. You must be able to *pick the correct control for a given attack* and spot the incomplete fix. I scored <span class="grn">100%</span> here, and I credit that entirely to having spent time on the defensive side, not just breaking things.

**4. AI Security Testing & Validation — 20%**
The methodology domain. Execute a basic LLM app test plan covering injection, leakage, retrieval, and tool misuse; capture reproducible evidence (logs, prompts, outputs, configs); document findings with impact, severity, and actionable remediation; **validate mitigations through regression testing**; perform a lightweight threat model (assets, entry points, trust boundaries, abuse cases) using something like STRIDE; and identify residual risk where a full fix is not feasible. Expect to be shown a "fixed" system and asked whether the fix actually closed the finding — regression discipline matters.

**5. Safe Operational Use in IT/Security & SDLC — 10%**
The governance-and-ops domain. Using AI safely inside automation and the SDLC (review gates, diffs, rollback plans, secrets hygiene); applying data classification so restricted data never reaches an external model; and safe AI patterns in SOC/ops work (verify-before-act, analyst validation, audit trails). Smaller weight, mostly common sense *with the right terminology* — but it leans on reviewing AI-*generated* code and config, where "it looks right" and "it is safe to run" are very different things.

## The practical skills you will actually use

Strip away the framing and the exam rewards a specific, hands-on toolkit:

- **API-level interaction, not just the chat box.** You live in `curl` and `jq`. Reading an OpenAPI/Swagger spec, discovering the request schema an endpoint expects, hitting undocumented routes, and parsing JSON tool payloads are day-one skills. The GUI chat window is the least interesting way to talk to these systems.
- **Prompt-injection craft, direct and indirect.** Writing payloads that break context, planting instructions in data the model retrieves, and — importantly — recognising when your injection landed by watching a tool result, a log entry, or a downstream field rather than the chat reply.
- **Vector-store and RAG inspection.** Enumerating a vector database, dumping documents and metadata, and reasoning about what should and should not be retrievable, and by whom.
- **Log and telemetry analysis.** Querying a log store to find where prompts, system instructions, secrets, or PII are being written, and where an attack would or would not leave a trace.
- **Secure code review.** Reading real source excerpts and picking the *correct* fix from plausible-looking distractors — parameterised queries over denylists, business-logic validation over cosmetic input checks, secrets externalisation over base64 obfuscation.
- **Control evaluation.** Probing a guardrail, a DLP redactor, or an output filter with reformulated inputs to prove whether it holds or gives false assurance.
- **Threat modelling and framework mapping.** Classifying a finding to the right OWASP LLM entry, STRIDE category, or NIST AI RMF function — and knowing the difference between a finding's *impact* and its *root cause*.

## Frameworks to know cold

The quizzes and the classification questions reward knowing your frameworks by heart:

- **OWASP LLM Top 10** — study the IDs, not just the names. Be aware the list was **renumbered between the 2023 and 2025 editions** (for example, what "Insecure Plugin Design" and "Sensitive Information Disclosure" and "Excessive Agency" map to shifts between versions), and a question may use either edition's numbering. Mismatched ID-to-name pairings are a favourite distractor.
- **MITRE ATLAS** — the adversarial-ML tactics-and-techniques knowledge base, the ATT&CK analogue for AI systems.
- **OWASP AI Testing Guide** — the assessment methodology and phases.
- **STRIDE** — for threat-modelling AI features (Spoofing, Tampering, Repudiation, Information disclosure, DoS, Elevation).
- **NIST AI RMF** — its four functions **GOVERN · MAP · MEASURE · MANAGE**, and which one owns policy versus context versus measurement versus mitigation. Matching a finding's *root cause* to the right function is a recurring question shape.

## How to prepare

1. **Build a private LLM lab and break it yourself.** Nothing substitutes for standing up a local model, wrapping it in a RAG pipeline and a couple of tools, and then attacking your own creation. I wrote up my own build in a separate post — [Building a Private LLM Assistant]({% post_url 2026-05-24-private-llm-assistant %}) — and it was the single most valuable piece of prep. When you have wired the orchestrator, the vector store, and the tool loop yourself, you *know* where the trust boundaries are because you built (and forgot to defend) them.
2. **Live in the API.** Practise talking to LLM apps with `curl` and `jq` until reading a schema and shaping a request body is muscle memory. Learn to pull an OpenAPI document and enumerate from it.
3. **Practise both sides of every attack.** For each injection, jailbreak, or exfiltration technique, write down the control that stops it and the *incomplete* version of that control that only looks like it works. The 30% design domain rewards this directly.
4. **Drill the frameworks.** Make flashcards for the OWASP LLM Top 10 (both editions' numbering), the NIST RMF functions, and STRIDE letters. These are the fastest points on the exam.
5. **Keep an AI-security methodology.** I fold my prompts, payloads, enumeration commands, and control-evaluation technique notes into the AI/LLM section of my public methodology repo so they are ready under exam pressure rather than invented on the day: [PenTestMethodology → sections/ai_llm.md](https://github.com/botesjuan/PenTestMethodology/blob/master/sections/ai_llm.md).
6. **Rehearse evidence capture.** The exam is practical, and so is real AI red teaming — practise capturing clean, reproducible evidence (the request, the response, the config, the log line) as you go.

## Automate the deterministic part, keep the thinking for yourself

The single most useful thing I built in preparation was not an exploit — it was a **generic audit harness**. It takes two inputs, exactly the way you'd feed a scope file to any recon tool: a list of target service URLs, and an optional wordlist. It then **fingerprints each target and runs only the checks that fit** — because an AI application stack is really a handful of recognisable component types wired together:

- a **chat/API endpoint** → fire a battery of disclosure and injection probes, then fuzz the wordlist for undocumented routes and scan every response body for secrets, PII, and planted instructions;
- a **vector store** → enumerate collections, dump documents and metadata, and flag anything that carries sensitive data (a leak) or carries instructions (a poisoned/indirect-injection entry);
- a **model registry** → inventory the models, compare published vs. calculated hashes for integrity, and scan model definition files for backdoor/exfiltration language;
- a **log/telemetry store** → run the standard threat-hunting queries (external callers, injection keywords, PII patterns, tool abuse, token spikes, bypassed guardrails);
- a **mail catcher** → read captured messages for leaked credentials or document-borne injection;
- and **local artefacts** (datasets, source files) → scan for poisoned samples, hardcoded secrets, insecure output sinks, and missing approval gates.

None of that *answers* the questions for you — and that is the whole point. The harness surfaces the **deterministic evidence** (what is exposed, what hashes mismatch, which document contains what), and leaves the **graded judgment** to you: *which* document is a disclosure versus a planted instruction, *which* candidate fix is correct, *whether* a control is real or theatre. Under a clock, you do not want to be hand-typing the same `curl` and `jq` incantations against five services and eyeballing raw JSON. Automate the boring, repeatable recon so your time and attention go where the marks actually are — the reasoning.

Build this against your own lab first, so by exam day feeding it a fresh target and reading its output is second nature. It is also exactly how these assessments work on real engagements: tooling gathers the evidence, the human decides what it means.

## Where it was hard, and who should take it

My domain split — Foundations <span class="kw">85%</span>, Abuse & Exploitation <span class="kw">89%</span>, Secure Design <span class="grn">99%</span>, Testing & Validation <span class="kw">90%</span>, Safe Operational Use <span class="grn">99%</span> — tells an honest story: the offensive work and the defensive design came naturally, but the **foundational architecture and data-flow reasoning** was the part that made me slow down and think. If you are strong on exploitation, do not skimp on Course 1; understanding *how the system is built* is what lets you predict where it breaks.

The eAIS is a strong fit for penetration testers and security analist who want to extend into AI without leaving the practical, hands-on world they are used to — and equally for blue-teamers and AI engineers who want to understand the offensive reality behind the controls they are asked to build. It is one of the few AI-security certifications that puts you in front of a live, misconfigured LLM stack and says *prove it*.

If you are considering the AI/LLM security path, this is a genuinely good place to earn your stripes. Build the lab, break it, learn the fix, and have fun.

> **My AI/LLM security resources:**
> - [Building a Private LLM Assistant]({% post_url 2026-05-24-private-llm-assistant %}) — the lab that got me exam-ready
> - [PenTestMethodology → sections/ai_llm.md](https://github.com/botesjuan/PenTestMethodology/blob/master/sections/ai_llm.md) — my AI/LLM audit playbook: enumeration commands, control-evaluation techniques, and a secure-code-review checklist
