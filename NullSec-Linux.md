# 🐧 NullSec Linux v5.0

NullSec Linux is a security-focused Linux distribution built on Debian 13, featuring 135+ pre-installed security tools and specialized editions for different security domains.

## Overview

| Feature | Details |
|---------|---------|
| Base | Debian 13 "Trixie" |
| Kernel | 6.8 LTS (security hardened) |
| Desktop | XFCE + Wayland/Hyprland |
| Tools | 135+ security tools |
| Editions | 9 (4 standard + 5 specialized) |
| Architectures | 4 (AMD64, ARM64, RISC-V, Apple Silicon) |

## Editions

### Standard Editions

| Edition | Size | Description |
|---------|------|-------------|
| **Pro Install** | 3.2 GB | Full installation with all tools |
| **Pro USB** | 4.1 GB | Persistent USB with encrypted storage |
| **Live Stealth** | 2.8 GB | RAM-only, anti-forensics |
| **Minimal** | 1.5 GB | Core tools only |

### Specialized Editions

| Edition | Focus | Tools |
|---------|-------|-------|
| **Cloud Pentest** | Cloud infrastructure | AWS, GCP, Azure, K8s, Terraform |
| **AI/ML Security** | Machine learning | LLM red team, prompt inject, adversarial |
| **Hardware Hacking** | Physical devices | SDR, RFID, JTAG, glitch |
| **Automotive Security** | Vehicle systems | CAN, OBD-II, UDS, key fob |
| **Mobile Security** | Mobile devices | NullKia pre-installed |

## Architecture Support

| Architecture | Support | Notes |
|--------------|---------|-------|
| AMD64 | ✅ Full | Primary development platform |
| ARM64 | ✅ Full | Raspberry Pi 4/5, Pine64 |
| RISC-V | ✅ Full | VisionFive 2, HiFive |
| Apple Silicon | ✅ Native | M1/M2/M3 via Asahi Linux |

## Key Features

### Security Hardening

- Kernel lockdown mode
- AppArmor profiles for all tools
- Secure boot support (optional)
- Full disk encryption (LUKS2)
- Memory protection (ASLR, PIE)

### Privacy Features

- MAC address randomization
- Tor integration
- DNS over HTTPS
- VPN configuration templates
- Anti-forensics tools

### Desktop Environment

- XFCE (default, lightweight)
- Wayland support
- Hyprland tiling compositor
- Dark theme with NullSec branding
- Custom tool launchers

## Tool Categories

### Core Security (15+ tools)
- Vulnerability scanning
- Exploitation frameworks
- Post-exploitation
- Privilege escalation
- Persistence mechanisms

### Network Security (20+ tools)
- Port scanning (nmap, masscan)
- Packet capture (Wireshark, tcpdump)
- Traffic analysis
- Protocol fuzzing
- Wireless auditing

### Web Security (15+ tools)
- SQL injection (sqlmap)
- XSS testing
- Directory brute-force
- API testing
- WAF bypass

### System Analysis (18+ tools)
- Binary analysis
- Memory forensics
- Process monitoring
- Kernel analysis
- Log analysis

### Cryptography (12+ tools)
- Hash cracking
- Encryption analysis
- Certificate management
- Password auditing
- Key management

### Cloud Security (6 tools)
- nullsec-cloudaudit
- nullsec-k8sscan
- nullsec-awsrecon
- nullsec-gcphunt
- nullsec-azuresweep
- nullsec-terraform-scan

### AI/ML Security (5 tools)
- nullsec-llmred
- nullsec-promptinject
- nullsec-modelaudit
- nullsec-adversarial
- nullsec-datapoisoning

### Hardware Security (6 tools)
- nullsec-sdr
- nullsec-rfid
- nullsec-canbus
- nullsec-jtag
- nullsec-glitch
- nullsec-uart

### Automotive Security (4 tools)
- nullsec-carfuzz
- nullsec-obdii
- nullsec-uds
- nullsec-keyfob

## Directory Structure

```
/opt/nullsec/
├── bin/              # NullSec binaries
├── tools/            # Third-party tools
├── wordlists/        # Password lists
├── payloads/         # Exploit payloads
├── scripts/          # Helper scripts
├── docs/             # Documentation
└── configs/          # Configuration templates

/usr/share/nullsec/
├── wordlists/        # Shared wordlists
├── templates/        # Report templates
└── icons/            # Application icons
```

## Configuration

### Environment Variables

```bash
# Add to ~/.bashrc or ~/.zshrc
export NULLSEC_HOME=/opt/nullsec
export PATH=$NULLSEC_HOME/bin:$PATH
export WORDLISTS=/usr/share/nullsec/wordlists
```

### Tool Aliases

```bash
# Pre-configured aliases
alias ns='nullsec'
alias nk='nullkia'
alias nmap-quick='nmap -sV -sC -O'
alias nmap-full='nmap -sV -sC -O -p-'
```

## Updates

```bash
# Update NullSec tools
sudo nullsec-update

# Update system
sudo apt update && sudo apt upgrade

# Update individual tool
sudo nullsec-install --upgrade <tool>
```

---

*See [Tool Categories](Tool-Categories.md) for detailed tool documentation*
