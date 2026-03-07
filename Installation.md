# 📥 Installation Guide

This guide covers installing NullSec Linux and the NullKia mobile security framework.

## NullSec Linux Installation

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 64-bit dual-core | 64-bit quad-core |
| RAM | 4 GB | 8 GB+ |
| Storage | 40 GB | 80 GB+ SSD |
| Display | 1024x768 | 1920x1080 |

### Download

1. Visit [NullSec Linux Releases](https://github.com/bad-antics/nullsec-linux/releases)
2. Choose your edition:
   - **Standard** — Full toolkit (140+ tools)
   - **Cloud** — Cloud security focus
   - **AI/ML** — Machine learning security
   - **Hardware** — Hardware hacking
   - **Automotive** — Vehicle security

3. Choose your architecture:
   - `amd64` — Intel/AMD 64-bit
   - `arm64` — ARM 64-bit (Raspberry Pi 4+, etc.)
   - `riscv64` — RISC-V 64-bit
   - `asahi` — Apple Silicon (M1/M2/M3)

### Verify Download

```bash
# Import GPG key
gpg --keyserver keyserver.ubuntu.com --recv-keys B1F1881F70FB62A7

# Verify signature
gpg --verify nullsec-linux-5.0-amd64.iso.sig nullsec-linux-5.0-amd64.iso

# Verify SHA256
sha256sum -c nullsec-linux-5.0-amd64.iso.sha256
```

### Create Bootable USB

#### Linux
```bash
# Find USB device
lsblk

# Write ISO (replace /dev/sdX with your USB device)
sudo dd if=nullsec-linux-5.0-amd64.iso of=/dev/sdX bs=4M status=progress && sync
```

#### Windows
Use [Rufus](https://rufus.ie/) or [balenaEtcher](https://etcher.balena.io/)

#### macOS
```bash
# Find USB device
diskutil list

# Unmount
diskutil unmountDisk /dev/diskN

# Write ISO
sudo dd if=nullsec-linux-5.0-amd64.iso of=/dev/rdiskN bs=4m && sync
```

### Boot Options

1. **Live Mode** — Run from USB without installation
2. **Live Mode (Persistent)** — Save changes to USB
3. **Install** — Full installation to disk
4. **Install (Encrypted)** — Full disk encryption with LUKS

### Installation Steps

1. Boot from USB
2. Select "Install NullSec Linux"
3. Choose language and keyboard layout
4. Configure network (optional)
5. Partition disk:
   - **Guided** — Automatic partitioning
   - **Manual** — Custom partition layout
6. Set root password
7. Create user account
8. Select additional packages
9. Install bootloader (GRUB)
10. Reboot

### Post-Installation

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install additional tools
sudo nullsec-install cloud  # Cloud security tools
sudo nullsec-install aiml   # AI/ML security tools
sudo nullsec-install hw     # Hardware hacking tools
sudo nullsec-install auto   # Automotive security tools

# Configure NullKia
nullkia --setup
```

---

## NullKia Installation

NullKia can be installed on any Linux, macOS, Windows, or Android (Termux).

### Linux

```bash
# Clone repository
git clone https://github.com/bad-antics/nullkia.git
cd nullkia

# Install dependencies
./scripts/install-deps.sh

# Build
make build

# Install
sudo make install

# Verify
nullkia --version
```

### macOS

```bash
# Install Homebrew if needed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install go libusb

# Clone and build
git clone https://github.com/bad-antics/nullkia.git
cd nullkia
make build
sudo make install
```

### Windows (WSL2)

```powershell
# Enable WSL2
wsl --install -d Ubuntu

# In WSL2 Ubuntu
git clone https://github.com/bad-antics/nullkia.git
cd nullkia
./scripts/install-deps.sh
make build
sudo make install
```

### Android (Termux)

```bash
# Install Termux from F-Droid
pkg update && pkg upgrade
pkg install git golang

git clone https://github.com/bad-antics/nullkia.git
cd nullkia
make build-android
```

### Docker

```bash
# Pull image
docker pull badantics/nullkia:latest

# Run
docker run -it --privileged badantics/nullkia:latest

# Or build locally
git clone https://github.com/bad-antics/nullkia.git
cd nullkia
docker build -t nullkia .
docker run -it --privileged nullkia
```

---

## Troubleshooting

### Boot Issues

**Problem:** System won't boot from USB
- Check BIOS/UEFI settings for Secure Boot (disable if needed)
- Verify USB was written correctly
- Try different USB port

**Problem:** Black screen after boot
- Add `nomodeset` to kernel parameters
- Try different graphics mode

### Installation Issues

**Problem:** Disk not detected
- Check BIOS for AHCI mode
- Load additional storage drivers

**Problem:** Network not working
- Check hardware compatibility
- Try wired connection during install

### NullKia Issues

**Problem:** Device not detected
- Check USB permissions
- Install udev rules: `sudo cp udev/99-nullkia.rules /etc/udev/rules.d/`
- Reload udev: `sudo udevadm control --reload-rules`

---

*See [Troubleshooting](Troubleshooting.md) for more solutions*
