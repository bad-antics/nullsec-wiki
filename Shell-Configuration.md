# Shell Configuration Guide

This guide covers the shell configurations provided with NullSec Linux.

## Overview

NullSec Linux includes pre-configured shell environments optimized for security research and development.

## Supported Shells

- **Bash** - Primary shell with extensive customizations
- **Zsh** - Alternative shell with minimal secure config

## Installation

Copy the configuration files to your home directory:

```bash
# Bash
cp /etc/nullsec/shell/bashrc ~/.bashrc
cp /etc/nullsec/shell/profile ~/.profile

# Zsh
cp /etc/nullsec/shell/zshrc ~/.zshrc
```

## Features

### Custom Prompt

The NullSec prompt provides clear visual identification:

```
[NullSec]-[current_dir]╼ 
```

### Security Aliases

Pre-configured aliases for common security tasks:

| Alias | Command | Description |
|-------|---------|-------------|
| `nmap-quick` | `nmap -sV -sC -O` | Quick service/version scan |
| `nmap-full` | `nmap -sV -sC -O -p- -A` | Full comprehensive scan |
| `nmap-vuln` | `nmap --script vuln` | Vulnerability scanning |
| `ports` | `ss -tunapl` | List open ports |
| `conns` | `netstat...` | Active connections |

### Hash Utilities

Quick hash generation and verification:

```bash
# Generate hashes
hashcheck /path/to/file

# Output:
# MD5:    abc123...
# SHA1:   def456...
# SHA256: ghi789...
```

### Password Generation

Secure password generation:

```bash
# Generate 24-character password (default)
genpass

# Generate 32-character password
genpass 32
```

### Network Functions

Quick network diagnostics:

```bash
# Check if port is open
portcheck 192.168.1.1 22

# Scan local subnet
localscan

# Get HTTP headers
headers https://example.com

# SSL certificate info
sslinfo example.com
```

### File Operations

```bash
# Extract any archive format
extract file.tar.gz
extract file.zip
extract file.7z

# Calculate file entropy
entropy suspicious_file.bin
```

## Customization

### Local Overrides

Create `~/.bashrc.local` for personal customizations that won't be overwritten:

```bash
# ~/.bashrc.local
export MY_VAR="value"
alias myalias='my command'
```

### Environment Variables

Key environment variables:

```bash
export EDITOR=vim
export VISUAL=vim
export PAGER=less
export HISTCONTROL=ignoreboth:erasedups
```

## Color Scheme

The shell uses a consistent color scheme:

| Color | Usage |
|-------|-------|
| Red | Prompt brackets, errors |
| Cyan | Username/hostname |
| Green | Directory, success |
| Yellow | Warnings |

## History Configuration

- **Size**: 10,000 commands
- **File Size**: 20,000 lines
- **Deduplication**: Enabled
- **Timestamps**: Enabled

## Security Notes

1. **Umask**: Set to `027` for secure file creation
2. **History**: Ignores commands starting with space
3. **PATH**: Only includes trusted directories

## Troubleshooting

### Prompt Not Showing Colors

Ensure your terminal supports 256 colors:

```bash
export TERM=xterm-256color
```

### Aliases Not Working

Source the configuration:

```bash
source ~/.bashrc
```

## See Also

- [Security Tools Reference](Security-Tools.md)
- [Installation Guide](Installation.md)
