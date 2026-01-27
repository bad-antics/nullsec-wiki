# NullSec Security Tools Reference

This page provides documentation for the security tools included in NullSec Linux.

## Table of Contents

- [Cryptographic Tools](#cryptographic-tools)
- [Network Tools](#network-tools)
- [Process Analysis](#process-analysis)
- [Log Analysis](#log-analysis)
- [File Analysis](#file-analysis)

---

## Cryptographic Tools

### nullsec-crypto

Multi-purpose cryptographic utility for hashing, encryption, and key generation.

#### Commands

| Command | Description |
|---------|-------------|
| `hash <file>` | Generate MD5, SHA1, SHA256, SHA512 hashes |
| `verify <file> <hash>` | Verify file against hash (auto-detects type) |
| `encrypt <file>` | AES-256-CBC encrypt with PBKDF2 |
| `decrypt <file>` | Decrypt .enc file |
| `genkey [bits]` | Generate random key (default: 256 bits) |
| `genpw [length]` | Generate secure password (default: 24 chars) |
| `entropy <file>` | Calculate file entropy |

#### Examples

```bash
# Generate hashes for a file
nullsec-crypto hash /etc/passwd

# Verify a downloaded file
nullsec-crypto verify file.iso abc123...

# Encrypt sensitive data
nullsec-crypto encrypt secrets.txt

# Generate a strong password
nullsec-crypto genpw 32
```

---

## Network Tools

### nullsec-netrecon

Network reconnaissance and enumeration utility.

#### Commands

| Command | Description |
|---------|-------------|
| `local` | Show local network information |
| `scan <target>` | Quick host/port scan |
| `ping <subnet>` | Ping sweep subnet |
| `arp` | Show ARP table |
| `routes` | Show routing table |
| `dns <domain>` | DNS enumeration |
| `whois <domain>` | WHOIS lookup |
| `headers <url>` | HTTP headers |
| `ssl <host>` | SSL certificate info |
| `ports <host>` | Common port scan |

#### Examples

```bash
# View local network info
nullsec-netrecon local

# Scan a target
nullsec-netrecon scan 192.168.1.1

# Ping sweep a subnet
nullsec-netrecon ping 192.168.1.0/24

# Check SSL certificate
nullsec-netrecon ssl example.com
```

---

## Process Analysis

### nullsec-procmon

Process monitoring and security analysis tool.

#### Commands

| Command | Description |
|---------|-------------|
| `list` | List all processes |
| `tree` | Process tree view |
| `top [n]` | Top N processes by CPU |
| `mem [n]` | Top N processes by memory |
| `find <pattern>` | Find processes matching pattern |
| `info <pid>` | Detailed process information |
| `files <pid>` | Open files for PID |
| `net <pid>` | Network connections for PID |
| `suspicious` | Find suspicious processes |
| `hidden` | Check for hidden processes |

#### Examples

```bash
# Find nginx processes
nullsec-procmon find nginx

# Get detailed info on a process
nullsec-procmon info 1234

# Check for suspicious activity
nullsec-procmon suspicious

# Monitor memory usage
nullsec-procmon mem 20
```

---

## Log Analysis

### nullsec-logview

Security-focused log analysis utility.

#### Commands

| Command | Description |
|---------|-------------|
| `auth` | Analyze authentication logs |
| `failed` | Show failed login attempts |
| `success` | Show successful logins |
| `ssh` | SSH-specific analysis |
| `sudo` | Sudo command history |
| `kernel` | Kernel message analysis |
| `errors` | Find errors across logs |
| `timeline <hours>` | Events in last N hours |
| `users` | User activity summary |
| `ips` | IP address summary |
| `watch <file>` | Live monitoring |

#### Examples

```bash
# Analyze auth failures
nullsec-logview failed

# SSH login analysis
nullsec-logview ssh

# Events in last 12 hours
nullsec-logview timeline 12

# Live monitoring with highlights
nullsec-logview watch /var/log/auth.log
```

---

## File Analysis

### nullsec-entropy

Analyze file entropy to detect encrypted or packed content.

```bash
# Check file entropy
nullsec-crypto entropy suspicious_file.bin

# Interpretation:
# > 7.5 bits/byte: Likely encrypted/compressed
# > 6.0 bits/byte: High entropy (binary)
# > 4.0 bits/byte: Medium entropy
# < 4.0 bits/byte: Low entropy (text)
```

---

## See Also

- [Installation Guide](Installation.md)
- [NullSec Linux Overview](NullSec-Linux.md)
- [Home](Home.md)
