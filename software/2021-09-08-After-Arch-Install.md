---
layout: post
title: After Arch Install
---

# After Arch Install

## Resources
- [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [General Post-Installation Recommendations](https://wiki.archlinux.org/title/General_recommendations)

## Introduction
To install Arch you have to download Arch by preferably torrent, verify the checksum, and burn the
iso. Verify the installation medium by having the iso and the sig in the same directory and run,
```
gpg --keyserver-options auto-key-retrieve --verify /path/archlinux-version-x86_64.iso.sig
```
Then verify the checksum at the [Arch download page](https://archlinux.org/download/).

Burn the iso to a USB drive by,
```
sudo umount /dev/sdX (not having it mounted)
sudo dd if=/path/to/debian.iso of=/dev/sdX bs=4M && sync
```
After installing a minimal distro, such as Arch, there are a few things to configure.

## Post-installation

#### Add user
```
pacman -S sudo
useradd -m -g wheel magnus
passwd magnus
```
Uncomment your option to give your user sudo privilege,
```
vim /etc/sudoers
```

#### Window manager
```
sudo pacman -S xorg-server xorg-xinit xorg-xrdb
sudo pacman -S i3-gaps i3status rxvt-unicode urxvt-perls dmenu picom
sudo pacman -S noto-fonts noto-fonts-emoji noto-fonts-cjk
```
Copy config file to home directory,
```
cp /etc/X11/xinit/xinitrc ~/.xinit
```
At the end of the file comment out and add i3,
```
#twn &
#exec xterm
exec i3
```
Now you can launch i3 with,
```
startx
```

#### Set up ssh key to Github
To connect your computer to Github through an ssh key see documentation in Github settings,
```
sudo pacman -S openssh
sh-keygen -t ed25519 -C "your_email@example.com"
```
Press enter to accept default location ~/.ssh/id_ed25519.pub. Then add the public key to Github.

#### Import dotfiles with stow and git
```
sudo pacman -S stow git
git clone https://github.com/HanssonMagnus/dotfiles-stow.git
cd ./dotfiles
stow *
```
