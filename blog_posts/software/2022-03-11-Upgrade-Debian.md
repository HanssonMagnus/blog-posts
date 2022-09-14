---
layout: post
title: Upgrade Debian 10 to 11
---

# Upgrade Debian 10 to 11

## Resources
- [Debian 11: Upgrading from Debian 10](https://www.debian.org/releases/stable/arm64/release-notes/ch-upgrading.en.html)

## Introduction
I've been running Debian 10 with kernel 4.19 on my server for a long time. However, recently I've had some troubles
with it. Especially when running software in the development stage. I got the following error when
upgrading my Ethereum node,
```
mdbx_env_open:16183 Linux prior to 5.4 requires MDBX_WRITEMAP because of a flaw of unified
page/buffer cache.
```
This is related to the [github issue](https://github.com/erthink/libmdbx/issues/269) where the
problem has been reproduced on different kernels `<5.4`.

## Upgrading Debian 10 (Buster) to 11 (Bullseye)
```
cat /etc/debian_version
```
yields `10.11`.

When upgrading to Debian 11 it's recommended to follow the
[official steps](https://www.debian.org/releases/stable/arm64/release-notes/ch-upgrading.en.html).
An overview of the steps,

- Backup your config data.
- Change `buster` to `bullseye` `/etc/apt/sources.list`. Note that `buster/updates` has changed to
  `bullseye-security`.
- `sudo apt update` and `sudo apt upgrade`

now `cat /etc/debian_version` shows `11.2` and `uname -r` (kernel release) shows `5.16.0-4-amd64`.

## Problems after upgrading
You might have some dependency issues. E.g., I'm using Discord, where the two dependencies
[libappindicator1](http://ftp.debian.org/debian/pool/main/liba/libappindicator/)
and [libindicator7](http://ftp.debian.org/debian/pool/main/libi/libindicator/)
were not part of Debian 11 Bullseye so I had to install them manually with `dpkg -i`.
