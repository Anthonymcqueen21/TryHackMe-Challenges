# 🛡️ Warzone 1 - TryHackMe SOC Challenge

## 📋 Challenge Overview
- **Platform:** TryHackMe
- **Path:** SOC Level 1 Analyst
- **Difficulty:** Medium
- **Scenario:** Analyze network traffic from a malware infection involving MirrorBlast/TA505 threat actor

## 🎯 Objectives
- Analyze PCAP file for malicious network activity
- Identify Command & Control (C2) IP addresses
- Extract and analyze malicious MSI installers
- Identify persistence mechanisms and dropped files

## 🛠️ Tools Used
- **Wireshark** - Network traffic analysis
- **tcp.stream** - File extraction from PCAP
- **MSI Database Tables** - Windows Installer analysis

---

## 🔍 Key Findings

### Malicious IP Addresses (C2 Infrastructure)

| IP Address | Purpose |
|------------|---------|
| 169.239.128.11 | Initial C2 communication |
| 195.149.87.128 | Secondary payload delivery |
| 185.10.68.235 | Payload staging server |
| 192.36.27.92 | MSI download server (Apache/2.4.41) |

### Malware Analysis: 10opd3r_load.msi

- **Disguised as:** Google Chrome v92.0.4515.107
- **Product Code:** `{7DAD0B07-2406-4203-AE21-B31650B1B6AE}`
- **Upgrade Code:** `{A2F91B1E-5C5B-4BBC-85F0-16F8CCAD5E7E}`
- **File Size:** 561,152 bytes

### Dropped Files
