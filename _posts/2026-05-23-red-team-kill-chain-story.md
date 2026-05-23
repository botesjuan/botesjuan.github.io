<style>
  .kw  { color: #ffd700; font-weight: bold; }
  .red { color: #ff4444; font-weight: bold; }   /* danger/alert terms */
  .grn { color: #00ff88; font-weight: bold; }   /* success/owned terms */
  .prp { color: #7b2fff; font-weight: bold; }   /* tool names */
</style>

---
layout: post
title: "<span class="red">red team</span> Operator: A Kill Chain Story"
date: 2026-05-23
categories: [<span class="red">red team</span>, Kill Chain]
tags: [red-team, cobalt-strike, active-directory]
---

# <span class="red">red team</span> Operator, a Kill Chain Story

**Operator,** Juan Botes | 22 May 2026

## Chapter 1, The Briefing

Every engagement starts with a single question, *if someone wanted to reach the most protected crown jewel of data in an organisation, how would they?*

The answer begins long before anyone touches a keyboard.

The target organisation ran three separate Windows domains, each sitting behind on seperated networks. Locked rooms linked by secure corridors. The outermost domain, call it the **child domain**, was where employees worked day to day. Beyond child domain lived the **parent domain**, home to the corporate headquarters and its domain controllers. And buried at the very end, invisible to the rest of the network and reachable only through layers of authentication and trust, was the **secure vault**, a hardened vault holding the organisation's crown jewels.

The rules were clear. A full security monitoring stack was watching, Elastic EDR recording every process, Sysmon logging every pipe and thread, Windows Defender scanning every file. Getting caught would cost points. Getting caught on the wrong machine would end the engagement immediately.

The objective, obtain a single low privilege user account named **Angela**, and use it, through whatever gaps existed in the architecture, to reach a locked file server in the secure storage vault and prove access by writing a "flag" file to its hard drive. Proving the possibility of ransomware deployment.

The command and control (<span class="kw">C2</span>) communication path did not yet exist. The <span class="kw">C2</span> chain of malware beacons would have to be built, hop by hop, identity by identity, to reach the crown jewels.

## Chapter 2, Arming Up

Before a single connection was made to the target network, the <span class="red">red team</span> operator spent time at their own attacker workstation preparing. This preparation would be the difference between an attack that gets blocked in the first sixty seconds and one that completes without a single alert.

The problem was this, modern endpoint security does not just scan files for known bad signatures. It watches *behaviour*. It notices when a process spawns unexpected children. It flags communication channels that look like command and control frameworks. It reads memory patterns in running applications and compares them against libraries of known malware. Running a standard, unmodified <span class="red">red team</span> beacon against this environment would be like walking into a bank wearing a ski mask.

The operator's answer was patience and craftsmanship.

The beacon loader, the small program that would live inside the target's processes and take instructions from the operator, was rebuilt from source code. Its internal decryption routine was rewritten backwards, producing identical behaviour but a completely different compiled byte pattern. Defender's signature scanner, which compared file contents against a database of known bad code, found nothing familiar.

The PowerShell scripts that would inject the beacon into memory had two specific strings that Microsoft's AMSI scanner, the engine that reads scripts before they execute, had learned to recognise as malicious. Both strings were encoded into base64 fragments and reconstructed at runtime, so the scanner never saw the original words. One substitution replaced a memory copy function with a lower level Windows API call that achieved the same outcome with a completely different appearance.

The command and control (<span class="kw">C2</span>) framework, the platform through which the operator would issue every instruction, was configured to disguise its network traffic. To any firewall or network monitor blue team watching outbound connections, the beacons would appear to be making ordinary HTTPS requests to a legitimate looking public website. The named communication pipes inside Windows, which Sysmon watches closely, were renamed to match the naming conventions of common .NET diagnostic tooling.

Finally, every payload was run through a detection scanner before use. Nothing left the attacker's workstation until it was confirmed clean.

The toolkit was ready. The operator turned their attention to the target.

## Chapter 3, Getting Through the Front Door

**Angela** was a domain user, ordinary, unremarkable, with no special privileges beyond local <span class="grn">Administrator</span> rights on a her single Windows workstation. That was the assumed breach, the organisation had handed over her credentials as the starting point, representing the realistic scenario of a social engineering, phishing email or stolen password.

The workstation had AppLocker enabled, a Windows application allow listing policy that prevented arbitrary programs from running. Only files in certain trusted folders and signed by approved publishers were permitted to execute. Defender's real time protection was active and watching.

The operator began by reading the AppLocker rules carefully. Policies like this are written by humans, and humans make exceptions. Buried in the configuration were several Windows system folders, that appeared on the allow list by default. Files placed in these locations were treated as trusted without further scrutiny.

A single PowerShell command, perfectly legal in Windows, downloaded the custom beacon payload from the operator's Intranet server and saved it into the allowed folder. Then windows own binary, Microsoft signed Windows utility that exists for the legitimate purpose of running code inside DLL files, was used to launch beacon malware. From Defender's perspective, a trusted system process was loading a clean file from a trusted path.

The beacon malware connected home to the public <span class="kw">C2</span> framework team server.

Within moments, a blinking indicator appeared on the operator's <span class="prp">Cobalt Strike</span> console, a new session, running with full SYSTEM privileges inside the process on Angela's workstation. The first foothold was established. The door was open.

## Chapter 4, Borrowing a Face

**SYSTEM** privilege is very powerful, but it is also conspicuous in certain ways. More importantly, Angela's workstation was just the first room. The operator needed to move deeper, and that required the right identity for the next door.

A review of the running processes on the machine revealed something useful, another user had an active session on the same workstation. Her name was **Brooks**, a member of the local workstation <span class="grn">Administrator</span> group on the next machine along the chain. She was logged in, her session was live, and her Windows security token, the digital credential that Windows checks whenever she opens a file or accesses a network share, was sitting inside one of her running processes.

SYSTEM level processes are permitted to reach into other processes and borrow their tokens. This is not an exploit; it is a documented Windows capability used legitimately by certain system services. The operator used it to extract Brooks' token from her active session and assume her identity inside the beacon.

No password was guessed nor cracked. No authentication protocol was attacked using password spraying or brute force. Brooks simply happened to be present on the same machine, and her token was borrowed as quietly as lifting a keycard from a jacket left on a chair or pickpocket from a passer by not noticing.

The operator confirmed the new identity and tested network access to the next workstation. Brooks' credentials opened it without hesitation.

## Chapter 5, Sliding Through the Next Door

The next challenge was establishing a presence on the second workstation without making noise. The most obvious approach, deploying a new executable, creating a new Windows service, would leave behind event log entries that any decent SOC analyst reviewing the morning's alerts would notice. Windows Event ID 7045 marks the creation of a new service. SIEM log ingestion of windows audit events will be easy to identify and a well known red flag.

The operator chose a different route.

A technique called shell injection obfuscation that modifies the binary path of a *pre existing* Windows service rather than creating a new one. The service already existed; it was already trusted. The operator changed where Windows was told to look for its executable, temporarily pointing it to the beacon payload, started the service to trigger execution, then restored the original path. The event log recorded Event ID 7040, a service configuration change, rather than the much more suspicious 7045 event id. Subtle with **finesse**, but meaningful in an environment where blue team threat hunters and analysts know what to look at.

A second beacon came online, this one inside the second workstation. Brooks had delivered the <span class="red">red team</span> operator to the next floor.

## Chapter 6, Another Face, Another Key

Brooks had served her purpose, but she did not have access to the intranet host. For that, the operator needed someone with different credentials.

**Charles** was a member of the Intranet Admins group, a user whose role granted local <span class="grn">Administrator</span> access to the organisation's internal server. And just like Brooks before him, Charles had an active session on this workstation. His authentication token was available in a running process on the intranet server from when the last code change control updated the company internal Intranet application.

The operator borrowed Charles authentication token <span class="prp">ticket</span>. Same technique, different face, different place, nothing to notice.

With Charles' identity loaded into the new malware beacon, the intranet server was the next destination.

## Chapter 7, Into the Server Room

The same service modification technique moved the beacon from the workstation into the organisation's internal Intranet server. Using Charles' administrative credentials, the lateral movement completed cleanly, no new services, no suspicious child processes.

A <span class="kw">C2</span> beacon appeared on the Internal Intranet Server.

But this machine was not the destination. It was the key to the next one. Because of where the Intranet server live in the network and what Windows permissions it had been granted, it was in a unique position to reach the database server, and to do so using someone else's identity entirely.

## Chapter 8, The Art of Borrowed Trust

This chapter is where the story turns from straightforward movement into something more elegant.

The Intranet server had been configured with a Windows delegation setting called, **Trusted to authenticated for Delegation**. This is an intentional enterprise feature, Intranet applications often need to access a backend database on behalf of the user who is visiting the website. Rather than sharing every user's password with the Intranet server, Windows allows the Intranet server to request Kerberos authentication <span class="prp">tickets</span> *in the name of any user*, as long as the domain controller has been told to trust Intranet Server to proof it's identity in doing so.

The organisation's System <span class="grn">Administrator</span> Infrastructure Team had set this up for a legitimate purpose, following Microsoft best practice guides. The <span class="red">red team</span> operator was about to use this for a different purpose and abuse.

The Intranet server's own machine account held a Kerberos <span class="prp">ticket</span>, a cryptographic credential that Windows caches automatically for every domain joined computer. The operator extracted this <span class="prp">ticket</span> from Windows memory using a Beacon Object File, a technique that runs entirely within the existing beacon process without spawning any child processes or opening any sensitive handles that EDR/XDR watches and monitors.

Using that machine <span class="prp">ticket</span> and the delegation permission, the operator asked the domain controller, *issue me a Kerberos <span class="prp">ticket</span> for the database service, but in the name of* **David**.

David was a Database System <span class="grn">Administrator</span>, an account with full control over the organisation's database server.

The domain controller obliged. The <span class="prp">ticket</span> was cryptographically valid, issued by the real Kerberos infrastructure, signed with the real domain keys. The Database server had no way to know it was handed to someone who had never typed David's password nor ever known what his password is!

## Chapter 9, The Database Admin's Gift

Armed with a valid Kerberos authentication <span class="prp">ticket</span> in David's name, the <span class="red">red team</span> operator connected to the Database server with high database privileges. No username and password prompt. No VPN. Just a <span class="prp">ticket</span> that said "I am David, and I have permission to be here."

Microsoft Database Server includes a feature called Common Language Runtime (CLR) integration, the ability for a database <span class="grn">Administrator</span> to load compiled .NET code directly into the Database Server process and execute it as a stored procedure. It exists for legitimate data processing tasks. The operator had prepared a custom .NET library during the arming up phase way back in the beginning before engaging.

CLR was enabled on the Database instance. The library was loaded. The stored procedure ran. Blue team missed the behavior as normal.

Inside Database Server's own process, not a new executable, not a new file on disk, not a new network connection, the <span class="kw">C2</span> beacon payload executed. It communicated outward through an existing named pipe <span class="kw">C2</span> communication channel, linking back to the operator through the Intranet server beacon already in place.

The database server was now under control by the <span class="red">red team</span> operator.

## Chapter 10, From Guest to Owner

The beacon running inside the Database Server process held the identity of the service account that ran Database Server, call it `TypicalDBServiceAccount`. This account had no special domain privileges, but it held one dangerous local Windows right, `SeImpersonatePrivilege`.

This privilege, the right to impersonate any user who connects to the process, is a standard part of how Database Server and certain other Windows services operate. In the hands of an operator who knows what to do with it, it is a reliable escalation path.

The technique is called a potato attack, and the mechanics of it read like a confidence trick played on the operating system itself. A fake COM server was constructed on a local port. The exploit then tricked Windows, through a well understood coercion mechanism, into connecting to that fake server using SYSTEM level credentials. When database server connected, the account `TypicalDBServiceAccount` exercised its impersonation privilege and absorbed the SYSTEM identity from that connection.

A TCP payload had been staged in advance and was listening on that local port, waiting for the SYSTEM connection to arrive.

Internal to the database server the connection arrived. The <span class="kw">C2</span> beacon connected. A new session appeared in the operator's console, this one marked with the lightning bolt that indicated SYSTEM privileges on the database server.

The Database Server box was now fully owned, and accepting commands from the <span class="red">red team</span> operator. 

## Chapter 11, Finding the Right Face

SYSTEM on a database server is powerful, but the operator's next destination was the domain controller. For that, domain <span class="grn">Administrator</span> credentials were needed.

A survey of active sessions on the database server revealed what the operator was looking for. **Fransisca**, a member of the Domain Admins group in the child domain, she had authenticated to this machine. Her Kerberos <span class="prp">ticket</span> was cached in Windows memory, just as Brooks' and Charles' tokens had been before her on other machines, waiting to be duplicated quietly.

The operator extracted Fransisca's <span class="prp">ticket</span> cleanly, using a obfuscation techniques that called Kerberos APIs directly without ever touching LSASS, the sensitive credential storage process that EDR systems watch with eagle eyes and particular attention to script kiddies not knowing better finesse techniques.

With a domain admin Fransisca's <span class="prp">ticket</span> in hand, the path to the child domain controller was open. 

## Chapter 12, Creeping up to the Castle

The child domain controller, fell the same way everything else had, a quiet modification of an existing service, a beacon deployed without fanfare, a new session established. No high fives, no load flash bang through the door or loud grenades exploding, counter strike first person shooting.

Domain controllers are the nerve centres of an Active Directory environment. They hold, sign, and distribute every authentication credential in the domain. Whoever controls the domain controller, controls the domain.

Fransisca's domain admin credentials opened the doors. The operator walked in to the controller room.

But the child domain was not the destination. The parent domain was. And beyond that, the secure vault. 

## Chapter 13, Crossing the Border

In a Microsoft Active Directory forest, child domains trust their parents. Children learn from teachers, but trust parents. This is by design, it allows users in a subsidiary to access resources in the corporate parent without separate logins. The trust is built into the windows architecture.

What it also means is that if you control the child, you can reach the parent, even ransom blackmail them if your intend.

Fransisca's domain <span class="grn">Administrator</span> privileges on the Child Domain Controller gave the operator something extraordinarily valuable, the ability to ask the domain controller to share its most sensitive secret. Every single Kerberos <span class="prp">ticket</span> issued in the child domain, every login, every file access, every authentication, was ultimately signed with a cryptographic key belonging to one special account called the `MasterGrantingKeyAccount`. 
Whoever held that key could forge any <span class="prp">ticket</span> they wanted. The operator performed a routine-looking domain synchronisation request, the same replication process that domain controllers use every day to stay in sync with each other. The child domain controller handed over the key without complaint. The monitoring system logged an Event 4662, replication activity on a domain controller, but nothing out of the ordinary. Perhaps the blue team night shift glanced at it and assumed Fransisca was running her regular administrative tasks.

With the master key in hand, the operator sat quietly at the attacker's desktop and crafted what is known as a Golden <span class="prp">ticket</span>. This was a Kerberos authentication credential signed with the child domain's own genuine key, one that the operator wrote themselves. It declared the bearer to be the child domain's <span class="grn">Administrator</span>, but it contained one extra ingredient hidden inside, the group identifier of the **Special Admins** group from the parent domain.

This extra ingredient is what made the border crossing possible. Microsoft designed a feature called SID history, which allows a user's group memberships to travel with them when they authenticate across a trust boundary. The parent domain controller was built to honour group memberships declared inside a <span class="prp">ticket</span> coming from a trusted child domain. When the Golden <span class="prp">ticket</span> arrived at the parent domain's gates, the controller inspected the signature, recognised the child domain's valid cryptographic seal, read the group memberships inside, and waved the bearer through as a full enterprise <span class="grn">Administrator</span>. 
The security guard at the headquarters door had seen these kinds of credentials many times before. Everything looked legitimate. Nothing raised an eyebrow.

The operator stepped into the parent domain's headquarters with unrestricted access to every system it controlled.

## Chapter 14, The Lifeline

Every experienced operator knows that a beacon chain is only as strong as its weakest link. The current chain stretched from the attacker's machine, through the Intranet server, through the database server, through the child Domain controller, and now into the parent Domain Controller. Five hops and easy to break with too many dependence and chance that a blue team threat hunting will disrupt the chained communication back out the building. Any single failure, a rebooted machine, a changed service configuration, a network hiccup, and the entire <span class="kw">C2</span> chain would collapse resulting in loss of access for the <span class="red">red team</span> operator.

The vault was coming next, and the vault was the most sensitive part of the engagement. If the <span class="kw">C2</span> beacon chain failed there, there would be nothing to fall back on.

The operator planted a backup before going further and ensuring persistence to come back and continue in dark of another night while not being disturbed and the blue team night shift doing graveyard duty.

A DNS beacon, a variant of the implant that communicates entirely through DNS queries rather than HTTP or SMB, was prepared and uploaded to windows on the parent Domain controller. The <span class="kw">C2</span> beacon binary was renamed, to a genuine Windows debugging tool with a name that would not raise eyebrows in a directory listing. Its file timestamp was altered to match another so it appeared as old as the operating system itself.

A Windows Management Instrument (WMI) event subscription was configured, a legitimate Windows automation feature, to execute <span class="kw">C2</span> beacon binary whenever the machine's Group Policy refreshed. Group Policy refresh is a routine Operating System event that happens automatically every ninety minutes on every domain controller in an organisation. The operator waited patiently for the refresh to trigger the first callback.

A second, completely independent SYSTEM beacon appeared on the parent Domain Controller, communicating purely through DNS queries, with no named pipe and no dependency on the SMB chain that preceded it. If everything else failed, this beacon would remain persistent, keeping the controller room backdoor open to return.

The safety net was in place. The operator turned toward the secure storage vault.

## Chapter 15, The Forged Passport

The secure storage vault had no trust relationship with the parent domain, not directly. But it had something almost as useful, a special Active Directory group that existed in the parent domain and whose members were granted one way access into the vault. Call it the **vault Access group**, like giving your dog sitter a spare key to feed the dog while you are away on holiday, trusting them with just enough access to do the one thing they need to do.

If you were a member of that special group, the vault's domain controller would let you in. The operator was not a member. But they were about to carry a credential that said otherwise.

The same replication technique that had unlocked the child domain now unlocked the parent. The operator asked the parent domain controller to share its master signing key, the `MasterGrantingKeyAccount` hash that underpinned every authentication credential issued across the entire parent domain. The request looked no different from routine domain controller synchronisation. The key was handed over.

Now, with the parent domain's master key in hand, the operator crafted a second Golden <span class="prp">ticket</span>. This one declared the bearer to be the parent domain's own <span class="grn">Administrator</span>, and buried inside it was the group identifier of the vault Access group. The <span class="prp">ticket</span> never touched any target machine. It was built entirely on the attacker's desktop, signed with a stolen key, and ready to be presented at borders that did not expect forgery.

The forged passport went first to the parent domain controller. The controller inspected the cryptographic signature, found it mathematically valid, saw the vault Access group membership, and issued a referral <span class="prp">ticket</span> for the vault's domain, just as it would for any legitimate member of that group. The vault's own domain controller then received that referral and issued the final service <span class="prp">tickets</span>, one permitting file-system access to the jump host, one permitting service delivery. Both <span class="prp">tickets</span> were genuine, issued by real infrastructure, because the chain of signatures leading back to them was unbroken. The forgery was at the very beginning, invisible from where the vault's controller was standing.

With the file-access <span class="prp">ticket</span> confirming reach to the jump host, the service <span class="prp">ticket</span> was used to land a new beacon there, quietly and without fanfare, through the same service-modification technique that had moved the operator through every machine before it. This new session linked back not through the long SMB chain of compromised machines but directly through the resilient DNS beacon sitting on the parent domain controller, independent and persistent regardless of what happened to anything else.

The operator had entered the secure storage vault.

Walking quietly inside, the heightened defences were immediately apparent. This machine ran the most aggressive endpoint detection configuration in the entire engagement, and the operator knew it. Any technique that caused a new child process to appear on the machine, even the standard method for running security tools, would trigger immediate network isolation. The beacon would go dark, unreachable, and the engagement would be over with no way back. There was no second chance here. From this point forward, every single action would run inside the existing <span class="kw">C2</span> beacon thread, invisible to process monitors, leaving no child processes and no suspicious communication on the host itself.

## Chapter 16, Building the Tunnel

The vault's domain controller sat on a segmented network with no direct route from the outside networks nor public internet. This segment network only trusted a few select hosts. The attacker's desktop could not send packets to the segmented crown jewel network. But the jump host beacon could, as it was trusted with the keys to the boss level room.

A socks proxy was started inside the jump host beacon. Socks proxy is means to direct traffic from an outside host through another to reach normally unreachable destination. The proxy application on the attacker's desktop was pointed at the <span class="kw">C2</span> team server, configured to route all traffic destined for the secure storage vault's segmented network through the proxy and out via the jump host. From that moment, the attacker's PowerShell local window could send Active Directory queries directly to the vault's domain controller as naturally as if the two machines were on the same desk. Like working from home and actually trusted to work from home.

The tunnel was open and pivoting the couch up to friends apartment via the staircase was possible with many swings to get there. The final attack could begin.

## Chapter 17, The Last Lock

The target was Secure File Storage, the locked file server in the vault. The operator had no credentials, no password, no user account with access, and no direct path to the secure file storage. What they had was the trusted jump host, and the jump host had something more valuable than a password.

**Every domain joined Windows machine holds a Kerberos <span class="prp">ticket</span> for its own computer account.** This <span class="prp">ticket</span> is cached automatically by the operating system and refreshed every ten hours. It represents the machine's own identity in the domain, and it is always present. The operator extracted the jump host's machine <span class="prp">ticket</span> using a Beacon Object File that called Kerberos APIs directly, no LSASS handle, no process injection, nothing for EDR to detect. Not resulting in blue team code red priority one cybersecurity incident response team reaction dropping out of the roof to put <span class="red">red team</span> operating in cuffs.

Through the SOCKS proxy tunnel, a PowerShell window was opened on the attacker's own desktop, authenticated as the jump host machine account using a `runas /netonly` session. Inside that window, Rubeus, a Kerberos utility running entirely on the attacker's machine and not on any victim <span class="kw">C2</span> beacon directly local, requested an LDAP service <span class="prp">ticket</span> for the vault domain controller and injected it into the session. Active Directory modification commands could now run from the attacker's desktop, routed through the tunnel, as if the attacker's machine were in the secure storage vault.

The attack exploited a Windows feature called **Resource Based Constrained Delegation**. The mechanism is straightforward in concept, a computer can be told to trust another specific computer to impersonate any user when accessing its services. Once that trust is written into Active Directory, the trusted computer can use Kerberos protocol extensions to obtain a service <span class="prp">ticket</span> for any user, including <span class="grn">Administrator</span>, against the target machine.

The `Allowed To Act On Behalf Of Other Identity` attribute on Secure File Storage's computer object was writable by any authenticated user in the domain. This was the root cause misconfiguration. The operator wrote the jump host's identity into that attribute using standard Windows RSAT tools, telling the vault's Kerberos infrastructure, when jump host says it is acting on behalf of a user to access Secure File Storage, believe me.

The write completed, and the flag was planted, as a historical symbol of victory, territorial capture, the culmination of a grueling military campaign by the <span class="red">red team</span>.

Then, using the jump host's machine <span class="prp">ticket</span> and the Rubeus S4U (Service for User) protocol extension, running entirely on the attacker's desktop, through the tunnel, the operator requested a file access service <span class="prp">ticket</span> for the Secure File Storage, impersonating an <span class="grn">Administrator</span>. The vault's own domain controller issued the <span class="prp">ticket</span> without hesitation. Cryptographically valid. Genuinely signed. Completely legitimate in the eyes of every authentication check the file server would perform.

The last lock was open and the boss level room defeated without alerting the dogs and rather just feeding them quietly. 

## Chapter 18, Planting the Flag

The <span class="grn">Administrator</span>'s file <span class="prp">ticket</span> was injected into the jump host <span class="kw">C2</span> beacon. A directory listing of the file server's local drive came back cleanly, full access confirmed.

The file write command ran in process, inside the <span class="kw">C2</span> beacon thread, using PowerShell invoked without spawning indicators of compromise (IOC), so there were no child processes, no anomalous events, no alert. <span class="red">red team</span> operator celebrating in quiet of the night.

A text flag file appeared on crown jewel secure storage server for the most secured server in the organisation. This could be your retail customer database, PCI card data server, PII of all customers, or vendor accounts to be paid next financial cycle.

The engagement was complete once all <span class="kw">C2</span> beacon evidence, payloads, configuration changes reverted and cleanup of artifacts performed to not leave any trace for forensic investigators to find. 

## Epilogue, What the Scorecard Said

SOC Blue Defenders had not detected any malware. No suspicious named pipes communication traffic. No new suspicious services created. No LSASS credentials from memory dumping or handles opened. No process injection detected. The decoy service canary account was not tripped and spotted as a trap account. The domain honeypots was identified and left completely untouched.

The entire path, from Angela's ordinary workstation to the crown jewel secure file server, had been walked in stealth avoiding being caught. 

## Lessons to be learned for the Blue Team SOC Defenders

The entire attack chain described in this story exploited no zero-day vulnerabilities. Every technique abused legitimate Windows features, trusted system paths, and built-in authentication mechanisms. Only a single initial password for the compromised account on the foothold was known. That is the uncomfortable truth defenders must sit with, the tools to compromise a domain are often already inside the domain, waiting to be picked up.

Here is remedial suggestion to the blue team to improve there detection and response procedures based on this story.

- **Tier your privileged accounts strictly.** Fransisca was a Domain Admin with an active session on a database server. That single fact handed the entire child domain to the operator. Domain Admins should only ever log in to domain controllers directly. If an <span class="grn">Administrator</span> needs to manage a server, they use a dedicated jump workstation. When privileged accounts leave footprints on general-purpose machines, attackers collect them.

- **Monitor DCSync events from non-domain-controllers.** Replication requests from machines that are not domain controllers are a significant red flag. Windows Event 4662 combined with a replication GUID from a workstation or member server should trigger an immediate alert. Tools like Microsoft Defender for Identity and Elastic Security have detection rules specifically for this pattern — make sure they are turned on and tuned.

- **Audit delegation settings on your servers.** The intranet server's setting is what allowed the operator to impersonate the database <span class="grn">Administrator</span> without knowing their password. Review every computer account in Active Directory for unconstrained and constrained delegation. If a server does not have a documented business reason to impersonate users, remove the setting.

- **Review RBCD attribute write permissions.** The final lock fell because any authenticated user in the secure vault could write the sensitive attribute on computer objects. This is almost never intentional. Audit who has write access to that attribute across your domain, and restrict it to domain <span class="grn">Administrator</span>s only.

- **Strip SeImpersonatePrivilege from service accounts that do not need it.** The database service account escalated to SYSTEM because it held this privilege. Database Server needs to run as a low-privilege account or a Group Managed Service Account (gMSA) without this right wherever possible. Regularly audit which non-OS accounts hold this privilege.

- **Monitor WMI event subscription creation.** The operator's persistent DNS beacon on the parent domain controller survived because it was triggered by a WMI event subscription, which looks like routine system automation. New WMI subscriptions should be rare and reviewed. Windows Event 5861 and Sysmon Event 19, 20, and 21 capture this activity. Alert on any new subscription that was not created through your known change management process.

- **AppLocker is only as strong as its writable-path audit.** The initial foothold worked because Windows system folders that appear on AppLocker's allow-list were also writable by a standard user. Regularly audit the intersection of AppLocker-trusted paths and paths where non-<span class="grn">Administrator</span>s can write files. Any overlap is a bypass waiting to be used.

- **Watch for process tree anomalies, not just file signatures.** The payloads in this engagement passed every static scan. Defenders who rely on antivirus alone will not see them. EDR configured to alert on unexpected parent-child process relationships, binaries making outbound network connections, or services spawning internet browser apps, will catch what signatures miss.

- **Your honeypot canary accounts only work if someone is watching.** A Kerberoastable decoy service account was deliberately left in the domain as a trap. The operator spotted it and avoided it. Blue teams should ensure these canary alerts are wired to high-priority, immediate-response queues. If a canary fires, stop everything and treat it as a confirmed intrusion in progress.

- **Assume breach, and hunt for it.** Everything in this story could have been discovered earlier through proactive threat hunting rather than waiting for an alert to fire. Hunting for unusual Kerberos <span class="prp">ticket</span> patterns, unexpected LDAP modifications, new named pipe traffic from unusual processes, and service binary path changes goes a long way toward finding operators who are specifically trying not to be found.
