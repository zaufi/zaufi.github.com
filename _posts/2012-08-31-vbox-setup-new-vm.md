---
layout: post
title: "HOWTO: Make a new VM using VirtualBox command line"
description: "Step by step instruction to create a virtual machine using VirtualBox and command line"
category: administration
tags: [virtualbox, howto]
---
{% include JB/setup %}

### Absatact

Step by step instruction to create a virtual machine using VirtualBox and command line w/ Ubuntu inside.

# Create a new VM

Ok, lets go!

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

1. `--memory 2048` 2048M of RAM
2. `--cpus 2` 2 CPU w/ PAE, ACPI, HPET and IOAPIC
3. use modern CPU instructions for hardware virtualization support: `--hwvirtex on`,
   `--hwvirtexexcl on` and `--vtxvpid on`
4. no hardware 3D graphics acceleration required (cuz we'd like to install a server :) as well as audio support:
   `--accelerate3d off` and `--audio none` options
5. we want to emulate Intel ICH9 chipset and the only Intel PRO/1000 T Server (82543GC) network card bridged
   w/ physical wlan0
6. enable VirtualBox Remote Desktop Extension on port 5555, so we can connect the VM using an RDP client
   (like krdc) later


Now it is time to make a 10Gb "disk" (I gave a full path to desired place for image file,
cuz don't like to have heavy files in my home :) BTW, one may use `--basefolder /storage/soft/vbox` option
in the very first command to move all files related to this VM out of `$HOME` directory...

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

then detach the disk, and set a primary boot device to HDD:

    zaufi@gentop ~ $ VBoxManage storageattach ubuntu-12.04 --storagectl storage --port 2 --medium none
    zaufi@gentop ~ $ VBoxManage modifyvm ubuntu-12.04 --boot1 disk




# Setup just installed Ubuntu 12.04

To make shared folders work VirtualBox Guest Additions must be installed. First of all u have to "insert"
an ISO image into virtual CD drive:

    zaufi@gentop ~ $ VBoxManage storageattach ubuntu-12.04 --storagectl storage --port 2 --medium /usr/share/virtualbox/VBoxGuestAdditions.iso --type dvddrive

then boot into VM and mount it:

    root@ubuntu:/# mount /dev/dvd /media/cdrom/
    mount: block device /dev/sr0 is write-protected, mounting read-only

    root@ubuntu:/# ll /media/cdrom/
    total 48527
    dr-xr-xr-x 4 root root     2048 Aug 20 18:44 ./
    drwxr-xr-x 4 root root     4096 Sep  2 07:07 ../
    dr-xr-xr-x 3 root root     2048 Aug 20 18:44 32Bit/
    dr-xr-xr-x 2 root root     2048 Aug 20 18:44 64Bit/
    -r-xr-xr-x 1 root root      647 Aug 17  2011 AUTORUN.INF*
    -r-xr-xr-x 1 root root     6966 Aug 20 18:35 autorun.sh*
    -r-xr-xr-x 1 root root     5523 Aug 20 18:35 runasroot.sh*
    -r-xr-xr-x 1 root root  7699918 Aug 20 18:39 VBoxLinuxAdditions.run*
    -r-xr-xr-x 1 root root 20625408 Aug 20 18:47 VBoxSolarisAdditions.pkg*
    -r-xr-xr-x 1 root root 13626392 Aug 20 18:29 VBoxWindowsAdditions-amd64.exe*
    -r-xr-xr-x 1 root root   282968 Aug 20 18:21 VBoxWindowsAdditions.exe*
    -r-xr-xr-x 1 root root  7431960 Aug 20 18:22 VBoxWindowsAdditions-x86.exe*

**before** run `VBoxLinuxAdditions.run` from it, make sure u have the following packages installed:

    root@ubuntu:/# apt-get install -y dkms build-essential linux-headers-virtual

They are required to build kernel modules. Now u may run additions installer...

    root@entell:/media/cdrom# ./VBoxLinuxAdditions.run
    Verifying archive integrity... All good.
    Uncompressing VirtualBox 4.1.20 Guest Additions for Linux.........
    VirtualBox Guest Additions installer
    Removing existing VirtualBox DKMS kernel modules ...done.
    Removing existing VirtualBox non-DKMS kernel modules ...done.
    Building the VirtualBox Guest Additions kernel modules ...done.
    Doing non-kernel setup of the Guest Additions ...done.
    Starting the VirtualBox Guest Additions ...done.
    Installing the Window System drivers ...fail!
    (Could not find the X.Org or XFree86 Window System.)

Now turn off the VM and lets add some shared folders:

    zaufi@gentop ~ $ VBoxManage sharedfolder add ubuntu-12.04 --name 'soft-storage' --hostpath /storage/soft
    zaufi@gentop ~ $ VBoxManage sharedfolder add ubuntu-12.04 --name 'host-exchange' --hostpath /storage/tmp

In `/storage/soft` I have a collection of .deb packages shared between VMs and schroot'ed systems which I have
in my host gentoo system -- just to avoid redundand downloads when many systems require updates).
And `/storage/tmp` will be used to exchange files between host and guest systems.

Now boot it again and append the following to the end of `/etc/fstab`:

    host-exchange              /mnt/host                vboxsf  defaults  0 0
    soft-storage               /storage                 vboxsf  defaults  0 0
    /storage/ubuntu/archives   /var/cache/apt/archives  none    rw,bind   0 0

**NOTE** the FS name! It is `vboxsf` not a `vobxfs` %)

Make sure all specified mountpoints and directories are exist! The last line is a rebind of a `.deb`s
storage, so `apt` will use a shared collection (this also will reduce size of the VM image in a host system).
The alternative way is to create a file `/etc/apt/apt.conf.d/90shared-archives` w/ the following content:

    Dir::Cache::Archives "/storage/ubuntu/archives";

so u wouldn't need a rebind entry in the `/etc/fstab` :)

**ATTENTION** Make sure the user u r running VM from has write permissions to shared archives directory
(on a host system), so `apt` (inside the VM) would be able to create a lock file and download new .deb files.

Before `mount -a` (or reboot) u may clean content of `/var/cache/apt/archives` to save some space whithin the VM.

### Important Notes

1. After `clonevm` do not forget to remove `/etc/udev/rules.d/70-persistent*` inside of VM
2. After removing some packages they may leave config files in a system, so `dpkg-query --list` will show _rc_
   status for them. It may affect futher (re)installs w/ conflicts in configuration files.
   To remove them completely use `dpkg -P <package>`... (BTW, there is a lot of packages can be removed after
   _minimal_ Ubuntu install which are obviously useless in VM)

### Useful Helpers ###

I found the following scripts pretty useful:

* `bin/startvm`
   {% highlight bash %}
#!/bin/bash
#
# Script to start a VBox'ed VM
#
# Copyright 2012 by Alex Turbov
#

if [ -z "$*" ]; then
    select vm in `VBoxManage list vms | sed 's,"\([^"]*\)".*,\1,'`; do
        break
    done
else
    vm=$1
fi

if [ -n "$vm" ]; then
    echo "Going to start '$vm'..."
    nohup VBoxHeadless -s "$vm" --vrde=config > "/tmp/nohup-$vm.log" 2>&1 &
else
    echo "Nothing to do..."
fi
   {% endhighlight %}
* `bin/stopvm`
   {% highlight bash %}
#!/bin/bash
#
# Script to stop a running VBox'ed VM
#
# Copyright 2012 by Alex Turbov
#

if [ -z "$1" ]; then
    select vm in `VBoxManage list runningvms | sed 's,"\([^"]*\)".*,\1,'`; do
        break
    done
else
    vm="$1"
fi

if [ -z "$2" ]; then
    a="`basename $0`"
    case "$a" in
    poweroffvm|savestatevm|resetvm|resumevm|pausevm)
        action=${a%%vm}
        ;;
    *) # default action
        action='poweroff'
        ;;
    esac
else
    action="$2"
fi

if [ -n "$vm" ]; then
    echo "Sending '$action' to '$vm'..."
    VBoxManage controlvm "$vm" $action
else
    echo "Nothing to do..."
fi
   {% endhighlight %}
* also I have improved bash-completion for `VBoxManage` (I'll share it later...)
