# THM Invite Only - SOC Analyst Investigation

![Banner](banner.png)

## Challenge Brief

**Platform:** TryHackMe  
**Path:** SOC Analyst 1 - Threat Analysis Tools  
**Role:** L3 SOC Analyst at TrySecureMe  
**Objective:** Analyze escalated IOCs from an L1 analyst and produce actionable threat intelligence

---

## The Investigation

### Initial Escalation

My day started with an escalation from an L1 analyst who flagged two suspicious indicators during morning IR activities:

- **Flagged IP:** `101.99.76.120`
- **Flagged SHA256:** `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`

The team had recently deployed TryDetectThis2.0, a threat intelligence platform similar to VirusTotal. My task was to dig deeper into these indicators and understand the full scope of the threat.

---

### Phase 1: Hash Analysis

I began by searching the SHA256 hash in TryDetectThis2.0. The platform returned a clear match with the filename **`syshelpers.exe`** and file type **`Win32 EXE`**. So far, so good.

However, I quickly realized this wasn't a standalone threat. Navigating to the **Relations** tab, I discovered **Execution Parents** - files that had dropped this payload. Two parents emerged:

1. `361GJX7J` (PowerShell, dated 2025-05-08)
2. `installer.exe` (Win32 EXE, dated 2025-06-01)

*Note to self: The chronological order mattered here, and I initially confused file types with filenames when submitting my answer. Lesson learned - pay attention to column headers.*

The second parent, `installer.exe`, proved critical. I noted its hash for later investigation.

---

### Phase 2: Multi-Stage Payload Analysis

Switching focus to the dropped files from `syshelpers.exe`, I found **`AClient.exe`** in the Dropped Files section. This was another piece of the puzzle - a secondary payload downloaded from `bitbucket.org`.

Now came the critical pivot: I searched the `installer.exe` hash and examined its Dropped Files. Four malicious files appeared:

This confirmed a multi-stage malware delivery chain. The attacker wasn't relying on a single executable but a coordinated deployment of multiple components.

---

### Phase 3: Infrastructure Analysis

With file analysis complete, I turned to the IP address: `101.99.76.120`. Searching this in TryDetectThis2.0 revealed **Communicating Files** - eight distinct samples reaching out to this IP.

Here's where I hit my first major obstacle: **VM instability**. The TryHackMe virtual machine began constantly reloading, preventing me from accessing cached reports for the communicating files. Multiple hashes returned "No Results Found," stalling my investigation.

*Adaptation required. I switched to my host machine and conducted OSINT research.*

Through Google searches combining the IP, hash, and "AsyncRAT," I identified the malware family: **AsyncRAT** - a .NET-based remote access trojan. The communicating files consistently showed AsyncRAT indicators, confirming this IP as command-and-control infrastructure.

---

### Phase 4: Threat Intelligence Enrichment

The final questions required context beyond technical analysis. I needed the original threat report, attack methodology, and tools used.

Through OSINT, I located the Check Point Research report: **"From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery."**

This report revealed the full attack chain:

| Element | Finding |
|---------|---------|
| **Initial Vector** | Hijacked Discord invites |
| **Phishing Technique** | ClickFix (fake error messages prompting users to "fix" their system) |
| **Cookie Stealing** | ChromeKatz tool |
| **Final Payload** | AsyncRAT for persistent access |

The attackers abused Discord's trusted platform to redirect users to malicious infrastructure, employed social engineering to execute their dropper, and deployed AsyncRAT for ongoing system compromise.

---

## Key Lessons

### What Worked
- Systematic pivoting from hash → parents → dropped files → IP → communicating files
- Maintaining documentation of hashes for multi-stage analysis
- Adapting quickly when technical limitations (VM failures) occurred

### What Needed Improvement
- **Interface Familiarity:** I initially struggled with tab navigation in TryDetectThis2.0. Knowing that Execution Parents and Dropped Files both live under Relations would have accelerated my analysis.
- **VM Reliability:** When the VM failed, I lost time before switching to alternative methods. Having OSINT resources bookmarked beforehand would help.
- **Precision:** I confused file types with filenames in my first submission. Reading column headers carefully prevents errors.

### Technical Takeaways
- AsyncRAT campaigns frequently abuse legitimate platforms (Discord, Bitbucket) for initial delivery
- Multi-stage malware requires tracking parent-child relationships - missing the dropper means missing the full scope
- Threat intelligence platforms may have incomplete cached data; external OSINT validates findings

---

searchhost.exe
syshelpers.exe
nat.vbs
runsys.vbs

## Indicators Summary

| Type | Indicator | Context |
|------|-----------|---------|
| SHA256 | `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f` | syshelpers.exe (AsyncRAT payload) |
| IP | `101.99.76.120` | AsyncRAT C2 server |
| Domain | `bitbucket.org/updatevak/upd/downloads/` | Payload hosting |
| Tool | ChromeKatz | Chrome cookie theft |
| Technique | ClickFix | Social engineering |
| Platform | Discord | Initial redirect |

---

## Attack Chain Visualization

[Hijacked Discord Invite]
↓
[ClickFix Phishing Page]
↓
[User Downloads installer.exe]
↓
[Dropper Deploys:]
├── searchhost.exe
├── syshelpers.exe (AsyncRAT)
├── nat.vbs
└── runsys.vbs
↓
[C2 Communication: 101.99.76.120]
↓
[ChromeKatz Steals Cookies]
↓
[Persistent Access Achieved]

---

## Completion

All ten questions answered. Challenge completed successfully despite technical obstacles.

**Skills Demonstrated:**
- IOC analysis and pivoting
- Multi-stage malware investigation
- Threat intelligence platform navigation
- OSINT and report correlation
- Adaptation under technical constraints

---

*Investigation completed: SOC Analyst 1 - Threat Analysis Tools pathway*
