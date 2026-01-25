# 📱 NullKia v3.0 Mobile Security Framework

NullKia is a comprehensive mobile security framework supporting 18 manufacturers with advanced baseband, cellular, TEE, and BootROM research capabilities.

## Overview

| Feature | Details |
|---------|---------|
| Version | 3.0.0 |
| Manufacturers | 18 |
| Platforms | Linux, macOS, Windows, Termux, Docker |
| Language | Go (core), Python (tools), Rust (performance) |
| License | MIT |

## Supported Manufacturers

| Manufacturer | Chipset | Support Level |
|--------------|---------|---------------|
| Samsung | Exynos, Snapdragon | ⭐⭐⭐ Full |
| Apple | A-series, M-series | ⭐⭐⭐ Full |
| Google | Tensor | ⭐⭐⭐ Full |
| OnePlus | Snapdragon | ⭐⭐⭐ Full |
| Xiaomi | Snapdragon, MediaTek | ⭐⭐⭐ Full |
| Huawei | Kirin | ⭐⭐ Extended |
| Motorola | Snapdragon | ⭐⭐⭐ Full |
| LG | Snapdragon | ⭐⭐ Extended |
| Sony | Snapdragon | ⭐⭐⭐ Full |
| Nokia | Snapdragon, MediaTek | ⭐⭐ Extended |
| Nothing | Snapdragon | ⭐⭐⭐ Full |
| OPPO | Snapdragon, MediaTek | ⭐⭐⭐ Full |
| Vivo | Snapdragon, MediaTek | ⭐⭐ Extended |
| Realme | Snapdragon, MediaTek | ⭐⭐ Extended |
| ASUS | Snapdragon | ⭐⭐⭐ Full |
| ZTE | Snapdragon, MediaTek | ⭐⭐ Extended |
| Fairphone | Snapdragon | ⭐⭐ Extended |
| TCL | MediaTek | ⭐⭐ Extended |

## Key Features

### 📡 Baseband Security

| Tool | Target | Description |
|------|--------|-------------|
| `shannon-exploit` | Samsung Exynos | Shannon baseband exploitation |
| `qcom-modem` | Qualcomm | Snapdragon modem tools |
| `mtk-baseband` | MediaTek | Helio baseband access |
| `exynos-bp` | Samsung | Exynos baseband processor |
| `apple-baseband` | Apple | iPhone baseband research |

### 📶 Cellular Security

| Tool | Protocol | Description |
|------|----------|-------------|
| `5g-fuzzer` | 5G NR | 5G protocol fuzzing |
| `lte-scanner` | 4G LTE | LTE cell scanning |
| `esim-manager` | eSIM | eSIM profile management |
| `imsi-catcher` | SS7/Diameter | IMSI collection tools |
| `carrier-unlock` | All | SIM/carrier unlock utilities |

### 🔐 TEE & TrustZone

| Tool | Target | Description |
|------|--------|-------------|
| `trustzone-probe` | ARM TZ | TrustZone security research |
| `tee-extract` | TEE | Trusted Execution Environment tools |
| `secure-element` | SE | Secure Element access |
| `knox-bypass` | Samsung | Knox security bypass research |
| `secure-enclave` | Apple | Secure Enclave analysis |

### 💾 BootROM & Firmware

| Tool | Target | Description |
|------|--------|-------------|
| `bootrom-dump` | All | BootROM extraction |
| `firmware-extract` | All | Firmware unpacking |
| `qfil-tools` | Qualcomm | QFIL EDL mode tools |
| `mtk-brom` | MediaTek | MediaTek BootROM exploit |
| `nand-dumper` | All | NAND flash extraction |

## Usage

### Basic Commands

```bash
# List connected devices
nullkia devices

# Device information
nullkia info <device-id>

# Interactive shell
nullkia shell <device-id>

# Run specific tool
nullkia run <tool> [options]

# GUI mode
nullkia gui
```

### Device Detection

```bash
$ nullkia devices

ID          Model               Manufacturer    Status
────────────────────────────────────────────────────────
SM-G998B    Galaxy S21 Ultra    Samsung         Connected
Pixel 7    Pixel 7 Pro          Google          Connected
iPhone14,2 iPhone 13 Pro        Apple           Connected
```

### Manufacturer-Specific Tools

```bash
# Samsung
nullkia samsung knox-check <device>
nullkia samsung odin-mode <device>
nullkia samsung shannon-dump <device>

# Apple
nullkia apple checkm8 <device>
nullkia apple sep-dump <device>
nullkia apple baseband-info <device>

# Google Pixel
nullkia pixel tensor-dump <device>
nullkia pixel bootloader-unlock <device>

# Qualcomm devices
nullkia qualcomm edl-mode <device>
nullkia qualcomm qfil-read <device> <partition>
nullkia qualcomm modem-dump <device>

# MediaTek devices
nullkia mtk brom-exploit <device>
nullkia mtk preloader-dump <device>
nullkia mtk da-mode <device>
```

### Baseband Analysis

```bash
# Dump baseband firmware
nullkia baseband dump <device> -o baseband.bin

# Analyze baseband protocols
nullkia baseband analyze <firmware.bin>

# Monitor baseband traffic
nullkia baseband monitor <device> --protocol lte

# Fuzz baseband interfaces
nullkia baseband fuzz <device> --interface at
```

### Cellular Security

```bash
# Scan for cells
nullkia cellular scan --band lte

# eSIM management
nullkia esim list <device>
nullkia esim dump <device> -o profiles.json

# IMSI tools
nullkia imsi collect --interface sdr

# Carrier unlock
nullkia carrier check <device>
nullkia carrier unlock <device> --method oem
```

### TEE/TrustZone

```bash
# Probe TrustZone
nullkia tee probe <device>

# Extract secure world data
nullkia tee extract <device> -o tee_dump.bin

# Knox bypass (Samsung)
nullkia knox check <device>
nullkia knox research <device>

# Secure Enclave (Apple)
nullkia sep info <device>
nullkia sep research <device>
```

## Plugin System

NullKia supports custom plugins for extending functionality.

### Plugin Structure

```
plugins/
├── my-plugin/
│   ├── plugin.yaml       # Plugin manifest
│   ├── main.go          # Plugin entry point
│   ├── commands/        # CLI commands
│   └── lib/             # Libraries
```

### Plugin Manifest

```yaml
name: my-plugin
version: 1.0.0
author: your-name
description: Custom plugin description
manufacturers:
  - samsung
  - google
commands:
  - name: my-command
    description: Does something cool
    handler: commands/mycommand.go
```

### Creating a Plugin

```go
package main

import (
    "github.com/bad-antics/nullkia/sdk"
)

func init() {
    sdk.RegisterPlugin(&MyPlugin{})
}

type MyPlugin struct{}

func (p *MyPlugin) Name() string {
    return "my-plugin"
}

func (p *MyPlugin) Execute(ctx *sdk.Context) error {
    device := ctx.Device
    // Plugin logic here
    return nil
}
```

### Installing Plugins

```bash
# Install from GitHub
nullkia plugin install github.com/user/my-plugin

# Install local plugin
nullkia plugin install ./my-plugin

# List plugins
nullkia plugin list

# Update plugins
nullkia plugin update
```

## GUI Mode

NullKia includes an interactive GUI for visual security analysis.

```bash
# Launch GUI
nullkia gui

# GUI with specific device
nullkia gui --device <device-id>

# Web-based GUI
nullkia gui --web --port 8080
```

### GUI Features

- Device management dashboard
- Real-time baseband monitoring
- Visual firmware analysis
- Protocol decoder
- Report generation

## Configuration

### Config File

```yaml
# ~/.nullkia/config.yaml
default_output: ./output
log_level: info
parallel_jobs: 4

devices:
  auto_detect: true
  timeout: 30s

plugins:
  auto_update: true
  repository: https://plugins.nullkia.dev

gui:
  theme: dark
  port: 8080
```

### Environment Variables

```bash
export NULLKIA_HOME=~/.nullkia
export NULLKIA_OUTPUT=./output
export NULLKIA_LOG_LEVEL=debug
```

---

*See [Baseband](Baseband.md), [Cellular](Cellular.md), [TEE](TEE.md) for detailed documentation*
