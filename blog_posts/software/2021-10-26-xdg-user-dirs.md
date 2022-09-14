---
layout: post
title: xdg-user-dirs
---

# xdg-user-dirs

## Resources
- [xdg-user-dirs](https://www.freedesktop.org/wiki/Software/xdg-user-dirs/)
- [Arch wiki: XDG user directories](https://wiki.archlinux.org/title/XDG_user_directories)
- [Arch wiki: Update XDG user dir configuration](https://man.archlinux.org/man/xdg-user-dirs-update.1)

## xdg-user-dirs
`xdg-user-dirs` is a tool to help manage "well known" user directories like the desktop folder and
the music folder. It also handles localization (i.e. translation) of the filenames.
[freedesktop.org](https://www.freedesktop.org/) attempts to standardize directories. Some software
use the default values (paths) of `xdg-user-dirs` to create paths and locate files. This is
annoying if you are not using these paths.

To show an xdg directory,
```
xdg-user-dir [NAME]
```

Description from the manual,
```
xdg-user-dir looks up the current path for one of the special XDG user dirs.

       This command expects the name of an XDG user dir as argument. The possible names are:
           DESKTOP
           DOWNLOAD
           TEMPLATES
           PUBLICSHARE
           DOCUMENTS
           MUSIC
           PICTURES
           VIDEOS
```

## xdg-user-dirs config
`xdg-user-dirs-update` updates the current state of the users user-dirs.dir. If none existed before
then one is created based on the system default values at `~/.config'. The default config looks
like the following,
```
# This file is written by xdg-user-dirs-update
# If you want to change or add directories, just edit the line you're
# interested in. All local changes will be retained on the next run.
# Format is XDG_xxx_DIR="$HOME/yyy", where yyy is a shell-escaped
# homedir-relative path, or XDG_xxx_DIR="/yyy", where /yyy is an
# absolute path. No other format is supported.
#
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
```

To disable a directory, point it to the homedir. My config looks like the following,
```
XDG_DESKTOP_DIR="$HOME/"
XDG_DOWNLOAD_DIR="$HOME/temp/"
XDG_TEMPLATES_DIR="$HOME/"
XDG_PUBLICSHARE_DIR="$HOME/"
XDG_DOCUMENTS_DIR="$HOME/"
XDG_MUSIC_DIR="$HOME/media/music/"
XDG_PICTURES_DIR="$HOME/media/pictures/"
XDG_VIDEOS_DIR="$HOME/media/videos/"
```

Apparentlyit is important for some apps
(like [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1082717)) to use `/` after the path.

## Mozilla firefox creates Desktop/Downloads dir in ~
Firefox is using the freedesktop.org's standard using the `xdg-user-dirs` trying to standardize
paths, and creates ~\Desktop or ~\Downloads dirs. These changes to the configuration file of
`xdg-user-dirs` stops this.
