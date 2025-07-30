# 🧊 Immutable Windows Platform

A modular, resettable, enterprise-ready Windows architecture — inspired by NixOS, Fedora Silverblue, and immutable OS concepts.


## 🚀 Overview

This project transforms Windows into an **immutable operating system** using:

- Bootable **VHDX userland images**
- **Reset-on-reboot** persistence logic
- Full **host OS lockdown**
- Snapshot and version control
- CLI + GUI tooling for developers and IT

### Use Cases

- Dev environments & CI/CD agents  
- Education & training labs  
- Secure kiosks or shared terminals  
- Enterprise policy-controlled endpoints

## 🧩 Architecture

### 1. Immutable Boot via VHDX

- `WindowsBase.vhdx`: Sealed reference image  
- `WindowsWorking.vhdx`: Writable, resettable runtime  
- Boot entries managed via `bcdedit`:  
  - Entry A: **Immutable Mode** (launch VHDX)  
  - Entry B: **Standard Windows**

### 2. Reset-on-Reboot

- Each reboot resets the working VHDX:  
  - Deletes `WindowsWorking.vhdx`  
  - Recopies from `WindowsBase.vhdx`  
- Enables reproducibility and rollback

## 🔐 System Lockdown

### 3. Inside the VHDX (Userland)

- Group Policy & registry hardening:  
  - Disables Task Manager, CMD, Explorer, etc.  
- Loopback GPO policies allowed  
- Optionally install only essential apps

### 4. Host Windows (Outer Shell)

- Hardened Pro/Enterprise or RE image  
- BitLocker encryption & firewall  
- NTFS lockdown on system and VHD paths  
- Auto-login or shell override to enter VHD  
- AppLocker or WDAC policy enforcement

## 🖥️ Tools

### 5. GUI Tray App (WinForms)

- .NET-based toggle UI for:
  - Rebooting to Immutable Mode
  - Rebooting to Standard Mode
  - Resetting working image manually  
- Minimal, secure tray utility for users

### 6. CLI Script: `winvhd-cli.ps1`

```powershell```
winvhd-cli.ps1 -Action <action> [-SnapshotName <name>]

Actions:
  boot-immutable       # Set next boot to VHD mode
  boot-normal          # Set next boot to normal mode
  reset                # Delete & recopy working VHDX
  snapshot             # Save current working VHD state
  restore              # Restore a named snapshot
  list                 # Show snapshot list

## 🧠 Snapshots

- Stored in: `C:\VHD\Snapshots\`  
- Used for version control, testing, backups  
- Snapshots can be made from working VHD or restored at any time


## ☁️ Enterprise Integration

- Fully manageable via:
  - Active Directory GPO  
  - Microsoft Intune / Endpoint Manager  
  - Azure DevOps (for VHD distribution, resets)

- Compatible with:
  - Defender ATP  
  - Sysmon / SIEM  
  - BitLocker key escrow

## 📁 Repo Contents

This is a single-file project containing all reference material:

- `README.md`: **Complete guide, logic, and flow**  
  - Boot strategy  
  - Lockdown guidance  
  - CLI usage  
  - System structure

No other code or files are provided — integrate manually or build using this guide.


## 🛠️ Roadmap

- [x] Add EXE/MSI installer  
- [x] Chocolatey / WinGet package  
- [x] Scheduled reset service  
- [x] Web dashboard for VHD management  
- [x] PXE boot + VHD overlay support

## 💡 Credits & Inspiration

- [NixOS](https://nixos.org/)  
- [Fedora Silverblue](https://silverblue.fedoraproject.org/)  
- [Vanilla OS](https://vanillaos.org/)  
- [Windows RE + VHD boot](https://learn.microsoft.com/)

Built with PowerShell, .NET Framework, and native Windows components.


## 📄 License

MIT License — use, modify, or distribute freely for personal or organizational purposes.


> **Windows as a reproducible, hardened, and resettable platform — built from native tools.**
