# Shadow Trace Challenge Write-Up

## The Scenario
Middle of the night shift. Manager calls in panicking about a suspicious file on a user's machine called `windows-update.exe`. Classic social engineering - name it something people trust so they double-click without thinking. My job: figure out what this thing actually does and where it's trying to connect before it spreads.

## File Analysis: The Binary Deep Dive

**What I did:**
Opened the binary in PEStudio and immediately saw red flags. Empty version info (real Windows updates have Microsoft signatures), 64-bit architecture, and three networking libraries loaded (WS2_32.dll, urlmon.dll, WININET.dll). For something claiming to be a Windows updater, it was way too chatty.

Found the SHA-256 hash for threat intel lookups and extracted the main IOC: an embedded URL pointing to `http://tryhatme.com/update/security-update.exe`. Classic typosquatting - "tryhatme" instead of "tryhackme" - designed to fool people in a hurry.

**The setback:**
Wasted time trying to use PowerShell to parse PE headers and extract architecture info manually. Typed commands with underscores instead of hyphens, got empty outputs, kept hitting walls. The debrief called this out rightly - I was trying to manual-parse what PEStudio shows in a GUI panel instantly.

**What worked:**
Once I just used PEStudio's visual interface, everything clicked. Hash? Copy-paste from the properties panel. Architecture? Listed right there in the header. URL? Extracted from the strings section. The tool did exactly what it's designed to do.

## Alerts Analysis: Connecting the Dots

**What I did:**
Two alerts told the full infection story. PowerShell alert showed base64-encoded obfuscation hiding a download from `https://tryhatme.com/dev/main.exe`. Chrome alert used ASCII character codes (`104,116,116...`) to hide another payload at `https://reallysecureupdate.tryhatme.com/update.exe`. Both used encoding to slip past basic detection.

The file saved was `test.txt` - innocuous name, malicious content.

**The setback:**
Initially misread the character codes in the chrome alert and decoded the wrong URL. Tried multiple variations before getting the format right. Frustrating when you know what you're looking for but the exact string format matters.

Also kept trying to use PowerShell on a static website view that had no VM backend. Wrong tool for the environment.

## Key Takeaways

**What I learned:**
1. **Tool selection matters.** PEStudio exists for a reason - use it instead of reinventing the wheel with PowerShell one-liners.
2. **Encoding is the attacker's friend.** Base64, ASCII codes, typosquatting - they all serve the same purpose: avoiding detection long enough to execute.
3. **IOCs are the currency of SOC work.** File hashes, malicious URLs, saved filenames - these are what you take to threat intel teams and feed into detection rules.

**What I'd do differently:**
- Start with the GUI tools first, manual parsing only if those fail
- Pay closer attention to format hints (asterisk counts actually matter)
- Remember the environment - static sites don't have PowerShell, VMs do

**Bottom line:** Malware authors aren't magicians. They use encoding, social engineering, and fake domains. My job is to peel back those layers, extract the real IOCs, and shut down their infrastructure before they get further in.
