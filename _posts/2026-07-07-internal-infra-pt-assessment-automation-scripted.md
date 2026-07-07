---
layout: post
title: "Internal Infra PT Assessment Automation Scripted"
date: 2026-07-07
categories: [Infrastructure, Active Directory]
tags: [internal, active-directory, adcs, bloodhound, automation, python, offline, methodology]
excerpt: "How I turned the repeatable grind of an internal infrastructure and Active Directory engagement into an offline-capable, non-destructive, chainable Python script set, driven by a fixed list of assessment goals and safe-by-default at every stage."
---

<style>
  .kw  { color: #ffd700; font-weight: bold; }
  .red { color: #ff4444; font-weight: bold; }
  .grn { color: #00ff88; font-weight: bold; }
  .prp { color: #7b2fff; font-weight: bold; }
  .cyan{ color: #00d4ff; font-weight: bold; }
</style>

# Internal Infra PT Assessment <span class="cyan">Automation</span>, Scripted

Every internal infrastructure engagement starts the same way. You plug into a subnet you have never seen before, and for the first hour you are not hacking anything, you are just *asking questions*. What is alive? Which boxes are domain controllers? What speaks SMB, which shares are open, where are the certificate services, is there a way out to the internet? The findings come later, but only if the enumeration underneath them was thorough, ordered, and did not miss the boring host in the corner that turns out to hold the crown jewels.

That opening hour is the same on every job. So I scripted it.

> **Repository:** [github.com/botesjuan/Private-Internal-Infra-AD-Pentest-Scripts](https://github.com/botesjuan/Private-Internal-Infra-AD-Pentest-Scripts)

This post walks through a reusable, **offline-capable**, **non-destructive** Python script set for internal infrastructure and Active Directory engagements. It requires no internet onsite, it is customer-agnostic, and every stage shares one output tree so each script's results become the next script's input. It is not a framework and it is not trying to be Metasploit, it is the disciplined, safe version of the enumeration I would otherwise be doing by hand at 11pm on day one.

> *The tool does not find the bug. It just makes sure you never skip the step that would have.*

## Starting from goals, not tools

Before a single line of Python, there was a list. Not a list of exploits, a list of **questions the client actually pays me to answer** on an internal test. It lives in `Internal-Assessment-Goals.md` and it is the spine the whole script set hangs off:

- Host discovery, ports and services, plus <span class="cyan">SIEM</span>, EDR and antivirus defence recon, so I map the monitoring as well as the machines.
- Identify the AD **domain controllers**.
- Validate real **user accounts**.
- Obtain credentials safely, username-as-password and a **single** password spray only, never anything that risks lockout.
- File servers and <span class="prp">SMB</span> shares, NFS mount points.
- Internal web and API applications, Swagger content, login portals, management interfaces.
- Virtualization hosts, Windows and Linux servers, IOT devices.
- Microsoft <span class="prp">AD Certificate Services (ADCS)</span> enumeration, web enrollment, templates and their vulnerabilities.
- Internal SMTP and Exchange, database hosts of every flavour, cloud connectivity from inside, local AI/LLM services, printers and their default passwords.
- Coercion and NTLM relay targets, taken the **safe** way.
- Passive wireless audit, no deauth, no client attacks.
- C2 and egress proxy policy checks.

Every script in the repo exists to answer one or more of those goals, and nothing else. Building from the goal list rather than from a pile of favourite tools is what keeps the set focused and keeps the output aligned to what ends up in the report.

## Three design principles that never bend

An internal engagement is played on someone else's production network. Fragile IOT, printers that fall over if you look at them wrong, account lockout policies waiting to lock out the very users you validated. So the whole set is built on three rules that never bend.

**Offline.** Standard-library Python only, no auto-updates, no dependency on being able to reach the internet from the client site. Every tool it shells out to degrades gracefully if it is missing. What you tested with in the lab is exactly what runs onsite in an air-gapped segment.

**Safe.** Enumeration only. No DoS, no `DROP` or `DELETE`, no data modification, no aggressive NSE scripts. There is even a destructive-command guard in `lib/pentest_lib.py` that blocks obvious bad tokens as a backstop, so a fat-fingered flag cannot turn a read into a write.

**No lockouts.** Nothing brute-forces and nothing sprays unless I explicitly opt in with **both** `-spraypassword` **and** `--confirm-spray`. The <span class="prp">kerbrute</span> user-enumeration and AS-REP roasting deliberately do not increment `badPwdCount`. The default posture is anonymous-first, and the tool has to be *told twice* before it does anything that could touch an account.

Those three principles are not comments in a README, they are the reason I can run this on a real client network without holding my breath.

## The chaining flow

The stages are designed to feed each other. Discovery has to run first because everything downstream consumes its lists, then each subsequent stage reads the relevant per-service list and writes its own. Every stage emits one-item-per-line `.txt` files into a shared `output/` tree, so the whole thing chains without me copy-pasting IPs between terminals.

```
  1. internal_enum_discovery.py  ── writes ──▶ output/discovery/
                       live_hosts.txt, svc_<name>.txt, dc_candidates.txt,
                       web_endpoints.txt, member_servers.txt, ports_all.txt
   ┌──────────┬──────────┬────────────┬───────────┬───────────┬───────────┐
   ▼          ▼          ▼            ▼           ▼           ▼           ▼
 2. ad     3. web     4. common   6. relay    5. adcs     7. cloud    reports
   recon     enum       svc         coerce      mscs        discovery
   reads     reads      reads       reads       reads       egress +
   dc_cand   web_ep     svc_*.txt   svc_smb     dc_cand +   ADFS/Entra
   +ldap/krb +portals   +DB/print   signing     /certsrv    hybrid

 (standalone) 8. wireless_audit.py   passive Wi-Fi scan, run on its own
```

### Stage 1, discovery, the foundation everything else stands on
`internal_enum_discovery.py` is the map-maker. Run as root it gets ARP discovery, a SYN scan and OS detection, unprivileged it falls back to a TCP connect scan. The curated port set is not just the usual suspects, it reaches for AI/LLM inference ports (11434 Ollama, 1234 LM Studio, 7860), management and infra ports (2375, 5601, 631 IPP, 7474 neo4j, 8291 MikroTik Winbox, 50000 DB2/SAP), and for total certainty on an odd port you pass `-ports all` for a full 1–65535 sweep. Any open port that matches **no** known service bucket gets banner-grabbed read-only with netcat, so a service never slips through just because no dedicated tool covers its port. Scans are resumable, a cached nmap XML is reused unless `--force`, and `--max-rate 500 --timing 2` gentles it right down for fragile networks.

### Stage 2, Active Directory recon
`active_directory_recon.py` runs anonymous-first, SMB info, RID brute, anonymous LDAP, <span class="prp">enum4linux-ng</span>, <span class="prp">kerbrute</span> user validation and AS-REP roasting. Hand it a credential and it adds kerberoasting, authenticated LDAP, and a <span class="prp">rusthound-ce</span> BloodHound collection that can auto-import straight into a running <span class="prp">BloodHound CE</span> for attack-path review. Roast hashes are never cracked on the testing host, they are written out for offline <span class="prp">hashcat</span> on the GPU box (`-m 18200` for AS-REP, `-m 13100` for kerberoast). The `--username-as-password` check is a single attempt per account, paired line by line, not a brute force, and it is guarded by `--lockout-threshold`.

### Stages 3 to 7, the rest of the terrain
- **Web and API** (`web_app_services_enum.py`), <span class="prp">whatweb</span>, <span class="prp">wafw00f</span>, <span class="prp">sslscan</span>, a small-wordlist <span class="prp">gobuster</span> and curl, surfacing login portals **and** exposed unauthenticated LLM APIs (it lists the model inventory but never sends an inference prompt).
- **Common services** (`common_svc_tests.py`), null and anonymous checks across ldap, database, ftp, ssh, vnc, web, nfs, smtp, telnet, smb and winrm, with `--check-defaults` pulling in SecLists per-service default-credential lists, capped to stay ban-safe, and SMB/WinRM/LDAP deliberately held to blank-password-only so a bulk list never risks AD lockout.
- **ADCS** (`mscs_look.py`), finds CA servers, `/certsrv` web enrollment, and with creds the vulnerable templates ESC1 through ESC8 via <span class="prp">certipy</span>, enumeration only, no request or forge.
- **Coercion and relay** (`coerce_relay_check.py`), this is goal 15 done the **safe** way. It identifies NTLM relay targets (SMB signing not required) and coercion candidates (PetitPotam, PrinterBug, DFSCoerce, WebDAV) by *reading signing policy and checking whether the Spooler/WebClient services are enabled*. It does not fire a coercion and it does not launch ntlmrelayx, it prints the exact authorised-only commands to run next.
- **Cloud** (`cloud_discovery.py`), maps egress to M365/Entra, AWS and GCP control planes, checks the link-local IMDS to see if the segment is cloud-hosted, and finds ADFS / Entra Connect / Exchange hybrid hosts. Read-only, no credentials sent, no metadata token retrieved.

### Stage 8, passive wireless
`wireless_audit.py` is strictly a normal station scan that associates with nothing. It flags Open/WEP/WPA1/TKIP, WPS-enabled APs and rogue SSIDs against an optional allowlist. **No deauth, no client attack, no handshake capture, no injection.** It sits out of the auto chain and runs on its own because it needs a Wi-Fi adapter.

## One command for the whole chain

The stages run individually, but the orchestrator `run_all.py` drives the lot in the right order. The workflow splits cleanly by what you have in hand.

**Phase 0, before you go onsite**, while you still have internet:
```bash
sudo ./preflight_check.py            # every tool + wordlist present? fix the reds NOW
```
Offline onsite means no second chance, so the preflight verifies every binary and wordlist up front.

**Phase 1, no credentials yet**, anonymous-first sweep:
```bash
sudo ./run_all.py -subnet 10.0.0.0/24 -domain corp.local -listusernames users.txt
```
Running under `sudo` buys the better host discovery, `-domain` lets kerbrute validate users and AS-REP roast lockout-safe, and `-listusernames` feeds the OSINT username list. In this workflow the AD stage automatically runs the lockout-safe **username==password** single test on every validated user, and any hit lands in `output/ad_recon/weak_credentials.txt` and at the top of `SUMMARY.md`. If it finds a login, `run_all.py` promotes that credential into the downstream stages by itself.

**Phase 2, the moment you have one credential**:
```bash
sudo ./run_all.py -subnet 10.0.0.0/24 -domain corp.local \
     -username jsmith -password 'P@ss' \
     --collect-bloodhound --bh-import --bh-pass 'BHpass'
```
That single command adds kerberoasting, authenticated enumeration, certipy ESC template analysis, credentialed service checks, and a rusthound-ce collection imported straight into BloodHound CE, ready for attack-path review.

**Phase 3, targeted follow-ups, only when authorised**, is where the guarded password spray lives, and it will not run until you supply the lockout threshold and the double confirmation.

## Everything is evidence

An internal engagement is only as good as the record you can hand back. Every external command the set runs is appended to `output/logs/commands.log` in the CLAUDE.md format, `[YYYY-MM-DD HH:MM] - [Tool] - [Purpose]` followed by the command itself. Every stage writes a `*_report.md`, and the orchestrator rolls the headline findings of all of them into a single `SUMMARY.md` you read first. The chainable `.txt` lists are both machine input for the next stage and human evidence for the report. When a High or Critical surfaces, a spray hit, an ESC template, an exposed LLM, the tool follows the same protocol I would, it flags it at the top of the summary so I confirm impact and notify the engagement lead before going any further.

## Why script the boring part

The senior tester is not distinguished by knowing more exploits, it is by never forgetting the boring step that becomes the critical finding. The unconstrained delegation you did not enumerate. The certificate template you skipped because it was 11pm. The writable share on the host nobody was looking at.

Scripting the enumeration does not replace the operator, it frees the operator. The machine sweeps the subnet, chains the lists, logs every command and rolls up the summary, and I spend my attention on the part that actually needs a human, deciding what the findings *mean* and where the real path to the crown jewels runs. The tool makes sure I never skip the step. The judgement is still mine.

> **Get it here:** [Private-Internal-Infra-AD-Pentest-Scripts](https://github.com/botesjuan/Private-Internal-Infra-AD-Pentest-Scripts)
