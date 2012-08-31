---
layout: post
title: "HOWTO: Make a new VM using VirtualBox command line"
description: "Step by step instruction to create a virtual machine using VirtualBox and command line"
category: administration
tags: [virtualbox, howto]
---
{% include JB/setup %}

Ok, lets go:

Create a fresh VM named *ubuntu-12.04* (just pure .vbox config file)

    zaufi@gentop ~ $ VBoxManage createvm --name ubuntu-12.04 --ostype Ubuntu_64 --register
    Virtual machine 'ubuntu-12.04' is created and registered.
    UUID: 785fda4f-1f06-4f92-ac0d-307c32f465a3
    Settings file: '/home/zaufi/.VirtualBox/Machines/ubuntu-12.04/ubuntu-12.04.vbox'

To check possible guest OS types one may try the following:

    zaufi@gentop ~ $ VBoxManage list ostypes

Setup VM and hardware:

    zaufi@gentop ~ $ VBoxManage modifyvm ubuntu-12.04 --memory 2048 --cpus 2 --pae on --acpi on --hpet on --ioapic on --hwvirtex on --hwvirtexexcl on --vtxvpid on --accelerate3d off --audio none --chipset ich9 --nic1 bridged --bridgeadapter1 wlan0 --nictype1 82543GC --vrde on --vrdeport 5555 --clipboard bidirectional

Here we requested:

1. ``--memory 2048`` 2048M of RAM
2. ``--cpus 2`` 2 CPU w/ PAE, ACPI, HPET and IOAPIC
3. use modern CPU instructions for hardware virtualization support: ``--hwvirtex on``,
   ``--hwvirtexexcl on`` and ``--vtxvpid on``
4. no hardware 3D graphics acceleration required (cuz we'd like to install a server :) as well as audio support:
   ``--accelerate3d off`` and ``--audio none`` options
5. we want to emulate Intel ICH9 chipset and the only Intel PRO/1000 T Server (82543GC) network card bridged
   w/ physical wlan0
6. enable VirtualBox Remote Desktop Extension on port 5555, so we can connect the VM using an RDP client
   (like krdc) later


Now it is time to make a 10Gb "disk" (I gave a full path to desired place for image file,
cuz don't like to have heavy files in my home :) BTW, one may use ``--basefolder /storage/soft/vbox`` option
in the very first command to move all files related to this VM out of ``$HOME`` directory...

    zaufi@gentop ~ $ VBoxManage createhd --filename /storage/soft/vbox/ubuntu-12.04-sda.vdi --size 10240
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Disk image created. UUID: 4f24c555-96ef-4781-8e79-76a461f75a77

Create an SATA controller (named *storage*) w/ 4 ports:

    zaufi@gentop ~ $ VBoxManage storagectl ubuntu-12.04 --name storage --add sata --controller IntelAHCI --sataportcount 4 --hostiocache off

and attach our disk image as HDD to it:

    zaufi@gentop ~ $ VBoxManage storageattach ubuntu-12.04 --storagectl storage --port 1 --medium /storage/soft/vbox/ubuntu-12.04-sda.vdi --type hdd

I've downloaded an Ubuntu 12.04 server CD before, now we can push it into a virtual DVD drive
and tell to Virtual Box to use it as first boot device:

    zaufi@gentop ~ $ VBoxManage storageattach ubuntu-12.04 --storagectl storage --port 2 --medium /storage/soft/ubuntu/ubuntu-12.04.1-server-amd64.iso --type dvddrive
    zaufi@gentop ~ $ VBoxManage modifyvm ubuntu-12.04 --boot1 dvd

Ready to start the VM and install an Ubuntu server from the DVD "inserted":

    zaufi@gentop ~ $ VBoxHeadless -s ubuntu-12.04 &
    Oracle VM VirtualBox Headless Interface 4.1.20_Gentoo_
    (C) 2008-2012 Oracle Corporation
    All rights reserved.

    VRDE server is listening on port 5555.

**NOTE:** Dunno about desktop version, but server has a special mode to install into a VM
(use F4 at boot screen to select it).

**ATTENTION:** Do not use anything than ext or xfs as root file system in case of minimal setup! Otherwise u'll
get an unbootable system! (cuz minimal kernel just have no other FS drivers in initrd)

To eject the DVD disk after installation, stop VM by typing:

    zaufi@gentop ~ $ VBoxManage controlvm ubuntu-12.04 poweroff

then detach the disk, and set primary boot device to HDD:

    zaufi@gentop ~ $ VBoxManage storageattach ubuntu-12.04 --storagectl storage --port 2 --medium none --type dvddrive
    zaufi@gentop ~ $ VBoxManage modifyvm ubuntu-12.04 --boot1 disk
