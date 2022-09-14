---
layout: post
title: Arch Maintenance
---

# Arch Maintenance

## Resources
- [ArchWiki system maintenance](https://wiki.archlinux.org/title/System_maintenance)
- [systemd/Journal](https://wiki.archlinux.org/title/Systemd/Journal)
- [Downgrading packages](https://wiki.archlinux.org/title/Downgrading_packages)

It is always advisable to read the latest updated ArchWiki articles.

## Introduction
Arch is a rolling release distro meaning that there are updates often every day. In order to have a
properly functioning system you need to maintain and update your system regularly. Although, it is
easy enough to run,
```
sudo pacman -Syu
```
to upgrade/update your system, there are best practices. Since, the Arch repo includes the latest
versions of programs, occasionally one can have issues with dependencies. E.g., one application
might require a version .X.2 of a program, but that program is now updated to .X.3.
Thus, there are scenarios where a system update can break your system.

## Best practices
- Before upgrading check [archlinux.org](https://archlinux.org/).
- Before upgrading kernel, xorg, systemd, etc check [bbs.archlinux.org/](https://bbs.archlinux.org/).
- Upgrade your system after important task have been done, such that you have time to fix potential
  errors.
- Always run pacman -Syu before installing a package.
- Read the cli messages/alerts when upgrading.
- You may have to reboot after upgrading, especially the kernel is difficult to patch without
  reboot.
- Be careful when using packages from the AUR. Avoid AUR helpers and check the package before
  installing.
- Upadate your [mirrors](https://wiki.archlinux.org/title/Mirrors) regularly. Fetch best mirrors
  for your region from [Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/).
- Clean up and sync your `~/.config` once in a while.
- If you want less frequent kernel upgrades you can install linux-lts (long term support).
- Check for Orphans `pacman -Qtd`, and remove them with `pacman -Qtdq | pacman -Rns -`.
- Backup your important data regularly.

## Problems and how to fix them
You can check if there are any failed systemd services with,
```
systemctl --failed
```
and you can look for errors in the logs located at /var/log and errors with systemd journal,
```
journalctl -p 3 -b
```
