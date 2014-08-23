---
layout: post
title: "No loader required anymore!"
description: "How to boot linux kernel with UEFI"
category: administration
tags: [howto]
---
{% include JB/setup %}


Few steps to boot your kernel from UEFI
---------------------------------------

1. make sure you have an `EFI System` type partition as a first one on your GPT partitioned drive  
    i.e. smth like this:

        root@gentop〉 ~〉 gdisk -l /dev/sdb
        GPT fdisk (gdisk) version 0.8.6

        Partition table scan:
        MBR: protective
        BSD: not present
        APM: not present
        GPT: present

        Found valid GPT with protective MBR; using GPT.
        Disk /dev/sdb: 117231408 sectors, 55.9 GiB
        Logical sector size: 512 bytes
        Disk identifier (GUID): AB335B72-84EE-4567-B776-C94F639DB796
        Partition table holds up to 128 entries
        First usable sector is 34, last usable sector is 117231374
        Partitions will be aligned on 2048-sector boundaries
        Total free space is 1887981 sectors (921.9 MiB)

        Number  Start (sector)    End (sector)  Size       Code  Name
        1            2048         1050623   512.0 MiB   EF00  EFI System
        2         1050624         2099199   512.0 MiB   8300  Linux filesystem
        3         2099200       115345407   54.0 GiB    8300  Linux filesystem

    it must be FAT32 formatted. (A lot of docs can be googled w/ details how to make it).
2. turn the following options in kernel's `.config`  

        CONFIG_EFI_PARTITION=y
        CONFIG_EFI=y
        CONFIG_EFI_STUB=y

3. compile your EFI ready kernel
4. copy kernel image (`bzImage`) to the `/EFI/` directory on the `EFI System` partition. Let it be
    just `/EFI/kernel.efi`.
5. for the final step you need to add a _Boot Entry_. To do so you have to have `sys-boot/efibootmgr` installed
    and your current kernel **must** be already loaded via UEFI :-) Internet has a lot of docs how to achieve that
    using bootable USB flash drive for example... So finally issue the following command:

        root@gentop〉 ~〉 efibootmgr -c -d /dev/sdb -l \\EFI\\kernel.efi -L 'Current Gentoo Linux' -u 'root=/dev/sdb3 ro'
        BootCurrent: 0002
        Timeout: 2 seconds
        BootOrder: 0002,0000,0001
        Boot0000* GRUB2
        Boot0001  Hard Drive
        Boot0002* Current Gentoo Linux

    `-u` option as you may notice used to pass additional params to the kernel. Alternatively,
    you may compile the kernel w/ that params built-in.

    As for me, I prefer to have the following boot entries:

    * _Current Gentoo Linux_ -- to boot into a last compiled kernel;
    * _Previous Gentoo Linux_ -- to boot into a previous kernel in case of troubles w/ the last compiled one;
    * _Stable Gentoo Linux_ -- a kernel which I update from time to time (approx after 3-4 releases) and known
         as stable and really working (in case of damn serious troubles ;-)

6. Now you can reboot, enter setup and/or use boot menu to choose (or override) default boot item (order),
    so the next boot your computer will automatically select just added entry and boot w/o any intermediate
    screens or questions (like boot loaders do). Actually using `efibootmgr` you can change boot order from
    linux as well (just read the `man` page).

<del markdown="1">... so now I'm thinking about to get rid of `grub2` from my system ;-)</del>
Upd: **DONE**
