---
layout: post
title: Manage Dotfiles with GNU Stow
---

# Manage Dotfiles with GNU Stow

## Resources
```
info stow
man stow
```

## Introduction
From the man-page:
Stow is a symlink farm manager which takes distinct sets of software and/or data located in
separate directories on the filesystem, and makes them all appear to be installed in a single
directory tree.

The program was initially used to administer compilation of programs such as Perl and Emacs when
the software packages was unorganized. Nowadays, package managers such as rpm, dpkg and Nix or
language based package managers such as pip or gems, are used for this task. However, GNU stow is
still used today for other purposes, e.g., to manage dotfiles.

Stow creates symlinks from a directory to the parent directory on your system, which when managing
dotfiles often are $HOME and $HOME/.config. Thus, you create your stow-dotfiles directory in $HOME.

From the man-page:
A "symlink" is a symbolic link. A symlink can be "relative" or "absolute". An absolute symlink
names a full path; that is, one starting from /.  A relative symlink names a relative path; that
is, one not starting from /.  The target of a relative symlink is computed starting from the
symlink's own directory. Stow only creates relative symlinks.

## Manage dotfiles with Stow
In order to use Stow for your dotfiles, you need to have them in a directory at $HOME with a structure that
Stow recognizes. In this directory you will have one subdirectory for each of your programs that
you have dotfiles for. Then Stow will create a symlink treating that directory as $HOME. Thus, if
you have a dotfile in ~/stow-dotfiles/bach/.bashrc a symlink will be created of .bashrc to $HOME and if you have a
dotfile in ~/stow-dotfiles/i3/.config/i3/config a symlink will be created to $HOME/.config/i3/config.

I have a git repo in my $HOME directory with the following structure:

```
.
├── bash
│   ├── .bash_aliases
│   └── .bashrc
├── i3
│   └── .config
│       └── i3
│           └── config
├── i3blocks
│   └── .config
│       └── i3blocks
│           └── config
├── ranger
│   └── .config
│       └── ranger
│           ├── commands_full.py
│           ├── commands.py
│           ├── rc.conf
│           ├── rifle.conf
│           └── scope.sh
├── vim
│   └── .vimrc
└── xresources
    └── .Xresources
```

## Invoking Stow
For Stow to create a symlink to the given location no other file with a similar name can be present
at that location. Thus, you need to delete or rename (e.g., config_old) already existing dotfiles.

To create a symlink for one program, say Vim, cd to the stow-dotfiles directory and,
```
stow vim
```

To create symlinks for all dotfiles in the directory,
```
stow *
```

## Simulation mode
If you want to see what symlinks Stow would make without making them,
```
stow --verbose --no *
```

## Stow works nicely with git
Dotfile management with Stow works very well with git. With version control you will be able to
track your changes and easily install a new machine with,
```
git clone my-git:dotfiles-stow.git && cd dotfiles-stow && stow *
```

Additionally, when you make changes to the symlinks on your computer the files in the directory
dotfiles-stow changes as well. This is very handy, and when you change
the configuration of your system you can simply commit and push the changes.

## Ignore certain files
If you don't want to symlink everything in the stow directory. This can be avoided by adding an ignore list.
Stow's default ignore list is,

```
# Comments and blank lines are allowed.

RCS
.+,v

CVS
\.\#.+       # CVS conflict files / emacs lock files
\.cvsignore

\.svn
_darcs
\.hg

\.git
\.gitignore

.+~          # emacs backup files
\#.*\#       # emacs autosave files

^/README.*
^/LICENSE.*
^/COPYING
```
The list can be found in [Stow's documentation](https://www.gnu.org/software/stow/manual/html_node/Types-And-Syntax-Of-Ignore-Lists.html).

If you put Perl regular expressions, one per line, in a '.stow-local-ignore' file within any top
level package directory, in which case any file or directory within that package matching any of
these regular expressions will be ignored.

If you by accident symlinked a directory, say .git, that you did not intend you can delete the
symlinks by cd to the stow-dotfiles directory and,
```
stow -D .git
```
You can also re-stow packages, i.e., first unstow and then stow again, this is useful for pruning
obsolete symlinks after an update,
```
stow -R vim
```
