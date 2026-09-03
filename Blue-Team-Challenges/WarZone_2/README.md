# WarZone 2 - Network Alerts Triage

[![TryHackMe](https://img.shields.io/badge/TryHackMe-WarZone%202-red)](https://tryhackme.com/room/warzonetwo)
[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)](https://tryhackme.com/room/warzonetwo)
[![Category](https://img.shields.io/badge/Category-SOC%20Analyst-blue)](https://tryhackme.com/room/warzonetwo)
[![Points](https://img.shields.io/badge/Points-300-green)](https://tryhackme.com/room/warzonetwo)

> **SOC Analyst Level 1 Challenge** - Investigate network alerts, analyze PCAP files, and confirm true positive malware infections.

---

## 📋 Table of Contents

- [Challenge Overview](#challenge-overview)
- [Tools Used](#tools-used)
- [Investigation Walkthrough](#investigation-walkthrough)
- [Indicators of Compromise (IOCs)](#indicators-of-compromise-iocs)
- [MITRE ATT&CK Mapping](#mitre-attack-mapping)
- [Key Findings](#key-findings)
- [Conclusion](#conclusion)

---

## 🎯 Challenge Overview

**Scenario:**  
You work as a Tier 1 Security Analyst L1 for a Managed Security Service Provider (MSSP). An alert triggered for:
- Misc activity
- **A Network Trojan Was Detected**
- **Potential Corporate Privacy Violation**

**Objective:** Inspect the PCAP and retrieve artifacts to confirm this alert is a true positive.

**Artifacts:**
- `Zone2.pcap` (7,715 packets)

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| [Brim](https://www.brimdata.io/) | Network traffic analysis and Suricata alert investigation |
| [Wireshark](https://www.wireshark.org/) | Deep packet inspection and DNS analysis |
| [NetworkMiner](https://www.netresec.com/?page=NetworkMiner) | File extraction from PCAP |
| [VirusTotal](https://www.virustotal.com/) | Malware analysis and threat intelligence |

---

## 🔍 Investigation Walkthrough

### Question 1: Trojan Alert Signature

**Question:** What was the alert signature for "A Network Trojan was Detected"?

**Answer:**
**Methodology:**
- In Brim, searched for `142.93.211.176`
- Examined DNS events → Identified queried domain

---

## 🚨 Indicators of Compromise (IOCs)

### Network IOCs
| Type | Indicator |
|------|-----------|
| **Malicious IP** | `185[.]118[.]164[.]8` |
| **C2 IP** | `64[.]225[.]65[.]166` |
| **C2 IP** | `142[.]93[.]211[.]176` |
| **Malware URL** | `awh93dhkylps5ulnq-be[.]com/czwih/fxla[.]php?l=gap1[.]cab` |
| **C2 Domain** | `a-zcorner[.]com` |
| **C2 Domain** | `knockoutlights[.]com` |
| **C2 Domain** | `safebanktest[.]top` |
| **C2 Domain** | `tocsicambar[.]xyz` |
| **C2 Domain** | `ulcertification[.]xyz` |
| **C2 Domain** | `2partscow[.]top` |

### File IOCs
| Type | Indicator |
|------|-----------|
| **Filename** | `gap1.cab` |
| **Filename** | `draw.dll` |
| **SHA256** | `3769a84dbe7ba74ad7b0b355a864483d3562888a67806082ff094a56ce73bf7e` |
| **SHA1** | `f3e9e7f321deb1a3408053168a6a67c6cd70e114` |
| **MD5** | `78e05075e686397097de69fb0402263e` |
| **Imphash** | `3f31bb7bb53b534359adef9d369abdc6` |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Execution | Command and Scripting Interpreter | [T1059](https://attack.mitre.org/techniques/T1059/) |
| Execution | Shared Modules | [T1129](https://attack.mitre.org/techniques/T1129/) |
| Persistence | Pre-OS Boot | [T1542](https://attack.mitre.org/techniques/T1542/) |
| Defense Evasion | Process Injection | [T1055](https://attack.mitre.org/techniques/T1055/) |
| Defense Evasion | Masquerading | [T1036](https://attack.mitre.org/techniques/T1036/) |
| Command and Control | Application Layer Protocol | [T1071](https://attack.mitre.org/techniques/T1071/) |
| Command and Control | Ingress Tool Transfer | [T1105](https://attack.mitre.org/techniques/T1105/) |

---

## 🔑 Key Findings

1. **Malware Family:** [Valak](https://www.virustotal.com/gui/file/3769a84dbe7ba74ad7b0b355a864483d3562888a67806082ff094a56ce73bf7e) (Banking Trojan)
2. **Detection Rate:** 55/67 security vendors on VirusTotal
3. **Delivery Method:** CAB file containing malicious DLL
4. **C2 Infrastructure:** Multiple IPs and domains using DGA patterns
5. **Evasion Techniques:**
   - Non-standard file extensions
   - Legitimate-looking user-agent strings
   - Use of "Not Suspicious" infrastructure
6. **Impact:** Banking credential theft, system compromise

---

## ✅ Conclusion

This investigation **confirmed a true positive** alert for a Valak malware infection.

**Attack Chain:**
1. Initial compromise via malicious CAB file download
2. Execution of `draw.dll` payload
3. C2 communication over multiple domains and IPs
4. Potential banking credential theft

**Lessons Learned:**
- Always correlate multiple alert types
- Enrich IOCs with threat intelligence
- Analyze both "suspicious" and "not suspicious" traffic
- Extract and analyze payloads from network captures

