---
name: arch
description: Arch Linux system management, packages, services, debugging, logs, storage, and desktop-adjacent administration
---

# Arch Linux Skill

## Principles
- Verify live state before changing system config.
- Prefer read-only diagnostics before remediation.
- Use Arch Wiki/current package docs when exact syntax or package names matter.
- Ask before package install/remove, service-impacting changes, bootloader work, or destructive storage operations.

## Package Management
```bash
pacman -Ss <term>          # search repos
pacman -Qi <pkg>           # installed package info
pacman -Ql <pkg>           # package files
pacman -Qo <file>          # owning package
sudo pacman -S <pkg>       # install; ask first
sudo pacman -Rns <pkg>     # remove; ask first
yay -S <pkg>               # AUR install; ask first
```

## Systemd / Logs
```bash
systemctl status <service>
systemctl --user status <service>
systemctl --failed
journalctl -b -p err
journalctl -u <service> -n 80
journalctl --user -u <service> -n 80
```

## Storage Safety
Before any disk, partition, filesystem, EFI, mount, format, clone, or bootloader operation:
```bash
lsblk -f
mount
df -h
```
Never rely on remembered NVMe numbering.

## Common Locations
```text
~/.config/          user configs
~/.local/share/     user data
~/.local/bin/       user scripts
/etc/               system configs
/var/log/           system logs
```

## Diagnostics
```bash
dmesg | tail -50
ps aux --sort=-%cpu | head
ss -tulpn
df -h
free -h
```
