---
layout: post
title: Systemd services
---

# Systemd Services

## Resources
- [Arch wiki: systemd](https://wiki.archlinux.org/title/systemd)
- [Wikipedia: init](https://en.wikipedia.org/wiki/Init)
- [Wikipedia: systemd](https://en.wikipedia.org/wiki/Systemd)

The man pages for `systemd`, `systemctl`, `systemd.unit` and `systemd.service` are useful.

## Introduction

## Systemd
Systemd is a common init (initialization) system for Linux. It runs as PID 1 and is thus the first
process that is started by the kernel. The init system is directly or indirectly the ancestor of
all other processes. Systemd is a set of programs for Linux with the primary function
to manage the system and its services. The Unix convention of adding a "d" to the end of daemons
gives Systemd its name. There is a bit of a controversy among Linux users where some regard Systemd
to be be bloated. However many like Systemd because of its fast, resource efficient, stable and
works out of the box. Systemd calls daemons "units" and there are 11 types of units. From the man
pages there are the following unit types,

1. Service units, which start and control daemons and the processes they consist of. For details, see
   systemd.service(5).

2. Socket units, which encapsulate local IPC or network sockets in the system, useful for
   socket-based activation. For details about socket units, see systemd.socket(5), for details on
   socket-based activation and other forms of activation, see daemon(7).

3. Target units are useful to group units, or provide well-known synchronization points during
   boot-up, see systemd.target(5).

4. Device units expose kernel devices in systemd and may be used to implement device-based
   activation. For details, see systemd.device(5).

5. Mount units control mount points in the file system, for details see systemd.mount(5).

6. Automount units provide automount capabilities, for on-demand mounting of file systems as well
   as parallelized boot-up. See systemd.automount(5).

7. Timer units are useful for triggering activation of other units based on timers. You may find
   details in systemd.timer(5).

8. Swap units are very similar to mount units and encapsulate memory swap partitions or files of
   the operating system. They are described in systemd.swap(5).

9. Path units may be used to activate other services when file system objects change or are
   modified. See systemd.path(5).

10. Slice units may be used to group units which manage system processes (such as service and scope
    units) in a hierarchical tree for resource management purposes. See systemd.slice(5).

11. Scope units are similar to service units, but manage foreign processes instead of starting them
    as well. See systemd.scope(5).

Units are named as their configuration files. Some units have special semantics. A detailed list is
available in systemd.special(7).

A unit file is a plain text ini-style file that encodes information about a unit (e.g, service,
socket, etc).
Along with a unit file foo.service, a "drop-in" directory foo.service.d/ may exist. All files with
the suffix ".conf" from this directory will be parsed after the unit file itself is parsed. This is
useful to alter or add configuration settings for a unit, without having to modify unit files.

Unit files are loaded from a set of paths determined during compilation that are mainly located
under `/etc/systemd/` and `/run/systemd/` (see man systemd.unit). Moreover, additional units might
be loaded into systemd ("linked") from directories not on the unit load path.

## Service unit
A unit file that ends with ".service" is of the type service and must include a [Service] section.

#### Example
Let's create a service, start it and monitor it. Let's create a shell script `~/temp/test.sh`,
```
#!/bin/bash
while true
do
    echo The time is $(date)
    sleep 1
done
```
make the script executable by `chmod +x test.sh`. We can test the script and see the output in the
terminal. Now `cd` to `/etc/systemd/system` and create a service file `test.service`,
```
cd /etc/systemd/system
sudo vim test.service
```
and edit it to,
```
[Service]
ExecStart=/home/magnus/temp/test.sh
```
Now we can enable the service with,
```
sudo systemctl start test.service
```
and we can monitor the service with,
```
systemctl status test
```
By default the standard output of a service goes to syslog so we can inspect the output with,
```
sudo tail /var/log/syslog
```
or by running,
```
sudo journalctl -u test -f
```
and we can stop the service with,
```
sudo systemctl stop test
```
This was a very simple service that we created and more options can be added to the service. These
options are listed in `man systemd.service` in the section named options. We could extend the
`test.service` file to include a description, to be started after `network.service`, a restart
option, etc, and to always start on boot up. Then the file could look something like,
```
[Unit]
Description=Test service displaying current time
After=network.service

[Service]
ExecStat=/home/magnus/temp/test.sh
Restart=always
WorkingDirectory=/home/magnus/temp
User=magnus

[Install]
WantedBy=multi-user.target
```

## System control
The `systemctl` command is the command that controls the systemd system and manages services.
To list all units,
```
systemctl list-units
```
and to list all services,
```
systemctl list-units | grep .service
```
To inspect a unit, e.g. `ssh.service` you can type,
```
systemctl status ssh.service
```
If you want to change or manipulate units you need to use `sudo`, e.g.,
```
sudo systemctl disable ssh.service
```

#### Enable a unit
Example from the man page systemd.unit,

The following snippet (highlighted) allows a unit (e.g.  foo.service) to be enabled via systemctl
enable:

```
[Unit]
Description=Foo

[Service]
ExecStart=/usr/sbin/foo-daemon

[Install]
WantedBy=multi-user.target
```

After running systemctl enable, a symlink /etc/systemd/system/multi-user.target.wants/foo.service
linking to the actual unit will be created. It tells systemd to pull in the unit when starting
multi-user.target. The inverse systemctl disable will remove that symlink again.

## Systemd journal
`journalctl` is the command for querying the systemd journal written by `systemd-journald.service`.
