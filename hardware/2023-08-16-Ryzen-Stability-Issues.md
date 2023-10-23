---
layout: post
title: AMD Ryzen 9 5950X Stability Issues on X370
---

# AMD Ryzen 9 5950X Stability Issues on X370

## Resources
- [X370-Pro Firmware](https://www.asus.com/ea/supportonly/prime%20x370-pro/helpdesk_bios/)
- [Ubuntu Server: Kernel crash dump](https://ubuntu.com/server/docs/kernel-crash-dump)

## Parts
* Motherboard: Asus Prime X370-Pro
* CPU: AMD Ryzen 9 5950X
* RAM: Vengeance LPX 64GB 3600MHz CL18 DDR4

## TLDR
Changing VSOC voltage to 1.0 and DRAM voltage to 1.35 solved the issue.

## Introduction
I've had some trouble with AMD processors for some time. I might have lost the silicon lottery
twice in a row or there is something fishy with the combination of CPU, RAM, MB, and BIOS settings
that I run. In 2020 I wrote the following in my Github Debian dotfile repo:

*I have actually had Debian 10 crash a couple of times, with the error messages regarding the CPU. I
do not know exactly what the error is, I have only heard that it has previously been problems with
AMD Ryzen CPUs and some kernels. The reason why I did not have any problems on Ubuntu, I think, is
beacuse I was running version 18, which ran a kernel where this bug wasn't present.*

At this point in time, I ran an AMD Ryzen 5 1600 with the Asus Prime X370-Pro motherboard. Before
installing Debian on this machine I ran Ubuntu and I did not report any issues, however I'm unsure
if I used the computer in the exact same way etc.

Furthermore, AMD Ryzen instability issues are widely known and for example discussed on the
[ArchWiki](https://wiki.archlinux.org/title/Ryzen).

I've since then changed the memory and the CPU and the error still occurs at my current setup with
the AMD Ryzen 9 5950X CPU.

## Kernel crash dump
Installing kernel crash dump to investigate the issue.
```
sudo apt install linux-crashdump
```
Select `yes` to let `kexec-tools` handle reboots and `yes` such that `kdump-tool` is enabled by
default. If you want to check that this is enables `nvim /etc/default/kexec` and look for
`LOAD_KEXEC=true` and `nvim /etc/default/kdump-tools` and check for `USE_KDUMP=1`.

Now we need to reboot the machine. Afterwards, we can see the current status of `kdump` with
`kdump-config show`,

```
DUMP_MODE:		kdump
USE_KDUMP:		1
KDUMP_COREDIR:		/var/crash
crashkernel addr: 0x
   /var/lib/kdump/vmlinuz: symbolic link to /boot/vmlinuz-6.2.0-33-generic
kdump initrd:
   /var/lib/kdump/initrd.img: symbolic link to /var/lib/kdump/initrd.img-6.2.0-33-generic
current state:    ready to kdump

kexec command:
  /sbin/kexec -p --command-line="BOOT_IMAGE=/boot/vmlinuz...
```

This tells us that the core dumps will be found in `/var/crash`.

## The Error
The computer runs smoothly without any issues when it's idle or when I run heavy calculations. I
have not turned off the computer since sometime during the spring, now it's September, and I've not
had any issues. However, when I run an Erigon Ethereum archive node (not maxing out the CPU) the
computer freezes randomly from time to time. It can run for a day or a few days and then all of a
sudden it freezes.

## Restricting the CPU to only C-state 1 (Did not work)
Adding the `processor.max_cstate=1`, `rcu_nocbs=0-11`, and `idle=nomwait` parameters is a way to
limit the CPU's sleep states, help to off-load RCU callback processing, and prevent the kernel from
executing the `MWAIT` instruction. This can sometimes improve system stability or performance for
certain specific tasks.

To limit a CPU to a certain C-state, you can pass the processor.max_cstate=X option in the kernel
line of `/etc/default/grub`. We can look at the current kernel parameters with `cat /proc/cmdline`.
Now lets `cd` to `/etc/default` and `sudo cp grub grub_backup`. Then add the following to `grub`:
`GRUB_CMDLINE_LINUX_DEFAULT="quit splash processor.max_cstate=1 rcu_nocbs=0-11 idle=nomwait"`. Now
we can updage grub with `sudo updage-grub` and reboot.

Now when we run `cat /proc/cmdline`,
```
BOOT_IMAGE=/boot/vmlinuz-6.2.0-33-generic root=UUID=dfae9aa3-c27e-4f1b-8535-94cac7dc19ba ro
quiet splash processor.max_cstate=1 rcu_nocbs=0-11 idle=nomwait vt.handoff=7
```
Here is a breakdown of what each parameter means:

- `BOOT_IMAGE=/boot/vmlinuz-6.2.0-33-generic`: This indicates the path to the kernel image that was
used to boot your system.

- `root=UUID=dfae9aa3-c27e-4f1b-8535-94cac7dc19ba`: This specifies the UUID of the root file
system.

- `ro`: Mounts the root filesystem as read-only initially. It will be remounted as read/write later
in the boot process.

- `quiet`: Suppresses most boot messages.

- `splash`: Shows the splash screen during boot.

- `processor.max_cstate=1`: This limits the CPU to a specific C-state for power management. Setting
it to 1 often helps in troubleshooting issues related to CPU power management.

- `rcu_nocbs=0-11`: This offloads RCU callback processing from all CPUs specified (from CPU 0 to
CPU 11 in this case).

- `idle=nomwait`: This disables mwait for CPU idle control, which can be useful in specific
performance or debugging scenarios.

- `vt.handoff=7`: This is related to the handoff between the bootloader and the kernel regarding
virtual terminal 7.

## Update BIOS
After updating the BIOS you can run `sudo dmidecode -s bios-version` to check the version. For me
the command now gives `6203`.

## What have not solved the issue
- Installing Debian
- Installing Ubuntu
- Changing memory.
- Changing CPU.
- Installing non-free firmware (AMD proprietary drivers).
- Updating BIOS.
- Manual overclocking at 1.175V and 44.5 on each CCX.
- Tweaking with the power supply to the CPU including overclocking.
- Disabling c-state control in the BIOS.
- Change to `Typical Current Idle` in BIOS/Advanced/AMD/CBS/Power Supply Idle Control.
- Restricting the CPU to only C-state 1 in the kernel parameters.

## Things to try
- Increasing VSOC voltage.
- Increasing DRAM voltage.

## Increasing VSOC and DRAM voltages
It could also be an issue of the CPU getting too low of a voltage when clocking down.
However technically, I've already tried to increase the VSOC voltage when overclocking without success. At
this point in time my hypothesis is as follows: I think that the initial issue was with the CPU and
that I fixed that, but that I also have a RAM issue.

The DRAM voltage is by default set to 1.2 voltage in the BIOS (for some reason?), I increased this
to 1.35 (which is the Corsair recommended voltage). I also changed the VSOC voltage from 0.975 to 1
voltages (do not go higher than 1.1).
For my motherboard these settings are changed in the BIOS, entering the advanced mode and
then Ai tweaker.

## Solution
In the end, after increasing the VSOC voltage to 1.0 and the DRAM voltage to 1.35 I had no more
crashes.
