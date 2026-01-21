# LuneOS Ubuntu PPA - Project Changelog

## Overview

This project creates an APT repository (PPA) to install LuneOS desktop environment on Ubuntu 24.04 (Noble).

**Repository URL:** https://github.com/Starkka15/luneos-ppa.git

---

## luneos-session Package

### Version 1.2.0 (Current)

The `luneos-session` package is a comprehensive configuration bundle that makes LuneOS work on Ubuntu.

#### What It Contains

```
luneos-session/
├── DEBIAN/
│   ├── control          # Package metadata
│   ├── postinst         # Post-installation script
│   └── prerm            # Pre-removal script
├── etc/
│   ├── luna-service2/   # LS2 hub configuration
│   └── palm/            # Palm/webOS configs
├── lib/systemd/system/  # Systemd service files (33 services)
└── usr/share/luna-service2/
    ├── api-permissions.d/      # API permission definitions
    ├── client-permissions.d/   # Client permission definitions
    ├── groups.d/               # Security group definitions
    ├── manifests.d/            # Service manifests (111 files)
    ├── roles.d/                # Role definitions
    └── services.d/             # Service definitions
```

**Total:** 629 luna-service2 config files, 33 systemd services, 365 /etc configs

---

## Issues Fixed

### 1. `source: not found` Error

**Symptom:**
```
./surface-manager.sh: 46: source: not found
```

**Root Cause:**
- `surface-manager.sh` uses `#!/bin/sh` shebang
- Script contains `source` command (bash-specific)
- Ubuntu's `/bin/sh` is dash, which doesn't support `source`

**Solution:**
postinst patches the shebang at install time:
```bash
sed -i '1s|#!/bin/sh|#!/bin/bash|' /usr/share/luna-surfacemanager/startup/surface-manager.sh
```

---

### 2. `LUNASERVICE ERROR -1027: Invalid permissions`

**Symptom:**
```
LUNASERVICE ERROR -1027: Invalid permissions for com.webos.lunasend
```

**Root Cause:**
- luna-service2 requires security configuration files
- These files define which services can talk to each other
- The debian packages from luneos-ubuntu didn't include these configs

**Solution:**
Copied ALL luna-service2 configs from LuneOS Yocto symlinks directory to luneos-session package:
- manifests.d/ - Service registration manifests
- roles.d/ - Service role definitions
- groups.d/ - Security group definitions
- api-permissions.d/ - API access permissions
- client-permissions.d/ - Client permissions
- services.d/ - Service definitions

---

### 3. Shell Scripts Not Executable

**Symptom:**
```
luna-surfacemanager.service: Main process exited, code=exited, status=203/EXEC
```

**Root Cause:**
Scripts may lose execute permission during package extraction.

**Solution:**
postinst ensures scripts are executable:
```bash
find /usr/share/luna-surfacemanager/startup -name "*.sh" -exec chmod +x {} \;
```

---

### 4. Display Manager Switching Broke Live Sessions

**Symptom:**
Using `systemctl mask gdm3.service` during package install could disrupt the user's current GDM session.

**Root Cause:**
Masking is immediate and aggressive - not suitable for live system changes.

**Solution:**
Use the proper Debian/Ubuntu display manager mechanism:
```bash
# Set the default display manager (read at boot)
echo "/usr/bin/luna-surfacemanager" > /etc/X11/default-display-manager

# Create systemd symlink (used by systemd at boot)
ln -sf /usr/lib/systemd/system/luna-surfacemanager.service \
       /etc/systemd/system/display-manager.service
```

This approach:
- Doesn't affect the current running session
- Only takes effect after reboot
- Is the standard way DMs like GDM, LightDM, SDDM handle switching

---

## postinst Script Actions

1. **Fix shebang** - Patches surface-manager.sh for bash compatibility
2. **Make scripts executable** - chmod +x on startup scripts
3. **Create /var/run/ls2** - PID directory for ls-hubd
4. **Create luneos group** - System group for LuneOS permissions
5. **Add users to luneos group** - All human users (UID >= 1000)
6. **Set display manager** - Configure luna-surfacemanager as default DM
7. **Enable services** - ls-hubd, db8, db8-maindb, connman

---

## prerm Script Actions (Uninstall)

1. **Disable LuneOS services** - luna-surfacemanager, ls-hubd, db8, connman
2. **Restore previous DM** - Detects and restores GDM/LightDM/SDDM
3. **Re-enable NetworkManager** - Ubuntu's default network manager

---

## Package Dependencies

```
Depends: luna-surfacemanager, luna-surfacemanager-conf,
         luna-service2-utils, libluna-service2, systemd
Conflicts: gdm3, lightdm, sddm
```

---

## Installation Instructions

### Add the PPA
```bash
# Add GPG key (if signed)
# curl -fsSL https://raw.githubusercontent.com/Starkka15/luneos-ppa/main/KEY.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/luneos.gpg

# Add repository
echo "deb [arch=amd64] https://raw.githubusercontent.com/Starkka15/luneos-ppa/main noble main" | \
  sudo tee /etc/apt/sources.list.d/luneos.list

# Update and install
sudo apt update
sudo apt install luneos-session
```

### After Installation
```
==============================================
LuneOS session installed successfully!

IMPORTANT: Reboot to start LuneOS desktop.
Your current session will continue normally.
==============================================
```

---

## Uninstallation

```bash
sudo apt remove luneos-session
# Reboot to return to GDM/Ubuntu desktop
```

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `/etc/X11/default-display-manager` | Tells Ubuntu which DM to start |
| `/etc/systemd/system/display-manager.service` | Systemd symlink to active DM |
| `/usr/share/luna-surfacemanager/startup/surface-manager.sh` | Main LuneOS startup script |
| `/var/run/ls2/` | PID files for luna-service hub |
| `/usr/share/luna-service2/` | All LS2 security configs |

---

## Git History

```
0af5962 Fix display manager switching to use proper Debian/Ubuntu mechanism
a3570aa luneos-session v1.2.0: Comprehensive config bundle
6d9fb9b Use systemctl mask instead of disable for GDM
6b65183 Fix Release date to Jan 19 2026
4e54e1b Fix Release date to avoid future date validation error
69cef55 Add luneos-session package with systemd services
```

---

## Architecture

```
Ubuntu 24.04 Desktop
    │
    ├── APT Repository (GitHub Pages)
    │   └── luneos-ppa/
    │       ├── dists/noble/
    │       │   ├── Release
    │       │   └── main/binary-amd64/Packages
    │       └── pool/main/
    │           └── *.deb packages
    │
    └── Installed Packages
        ├── luneos-session (config bundle)
        ├── luna-surfacemanager (compositor/DM)
        ├── luna-service2 (IPC bus)
        ├── ls-hubd (hub daemon)
        ├── db8 (database service)
        └── connman (network manager)
```

---

## Known Issues / TODO

1. **GPG Signing** - Repository is currently unsigned
2. **arm64 Support** - Only amd64 packages currently
3. **Testing** - Need to verify full boot sequence works
4. **Additional Services** - May need more LuneOS services enabled

---

## Source Repositories

- **LuneOS Ubuntu Packages:** `/home/stark/luneos-ubuntu/`
- **LuneOS Yocto (reference):** Contains original configs in symlinks/
- **PPA Repository:** `/home/stark/luneos-ppa/`

---

*Last updated: January 20, 2026*
