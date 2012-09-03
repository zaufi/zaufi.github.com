---
layout: post
title: "HOWTO: Make a chroot'ed Debian/Ubuntu"
description: "Step by step instruction to create a chrooted Debian/Ubuntu environment"
category: administration
tags: [chroot, howto]
---
{% include JB/setup %}

Ok, lets go!

First of all u have to emerge a ``debootstrap`` package, then become root and do the following:

    root@gentop ~ $ cd /storage/schroot/

    root@gentop /storage/schroot $ debootstrap oneiric oneiric.schroot
    I: Retrieving InRelease
    I: Failed to retrieve InRelease
    I: Retrieving Release
    W: Cannot check Release signature; keyring file not available /usr/share/keyrings/ubuntu-archive-keyring.gpg
    I: Retrieving Packages
    I: Validating Packages
    I: Resolving dependencies of required packages...
    I: Resolving dependencies of base packages...
    I: Checking component main on http://archive.ubuntu.com/ubuntu...
    <...skipped...>

This command will prepare base environment for Ubuntu 11.10 (to make same for 12.04 just replace
``oneiric`` to ``precise``). The ``/storage/schroot/`` is a base directory where I keep all my chroot'ed systems.

Next step is to install ``schroot`` and write a config files for it (``/etc/schroot/schroot.conf``):

    [oneiric]
    type=directory
    description=Ubuntu Oneiric 11.10
    directory=/storage/schroot/oneiric.schroot
    users=build
    groups=users
    root-groups=root,wheel
    script-config=ubuntu/config

The last line points to a config file relative to ``/etc/schroot`` w/ the following content:

    # Filesystems to mount inside the chroot.
    FSTAB="/etc/schroot/ubuntu/fstab"

    # Files to copy from the host system into the chroot.
    COPYFILES="/etc/schroot/ubuntu/copyfiles"

    # System NSS databases to copy into the chroot.
    NSSDATABASES="/etc/schroot/ubuntu/nssdatabases"

The most intersting here is a ``fstab``:

    /proc     /proc     none    rw,bind     0   0
    /sys      /sys      none    rw,bind     0   0
    /dev      /dev      none    rw,bind     0   0
    /dev/pts  /dev/pts  none    rw,bind     0   0
    /tmp      /tmp      none    rw,bind     0   0
    /home     /home     none    rw,bind     0   0
    /storage/soft/ubuntu/archives   /var/cache/apt/archives none    rw,bind     0   0

Other files could be taken from default configuration (``/etc/schroot/default``).
Alto it would be nice to have the following function in a ``~/.bashrc``:

    function build-chroot()
    {
        set_misc_konsole
        cd /home/builder
        select c in `/usr/bin/schroot -l`; do
            /usr/bin/schroot -c $c -u builder
            break
        done
        set_normal_konsole
    }

so u may just run it from the bash prompt to switch into a chroot'ed environment of your choice
(as user _builder_, so do not forget to add such user or use some other name).
