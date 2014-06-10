---
layout: post
title: "HOWTO: Make a chroot'ed CentOS"
description: "Step by step instruction to create a chrooted CentOS environment"
category: administration
tags: [chroot, howto, centos]
---
{% include JB/setup %}

Unfortunately there is no anything [similar to `debbotstrap`]({% post_url 2012-09-02-howto-make-a-debian-chroot %})
package for RPM based distros in Gentoo, so some sort of manual work is inevitable.

Ok, lets go!

First of all we need `rpm` and `yum` programs to be installed (yeah, you can `emerge` them).
Then we can initialize RPMs database (I'll to my chroot environment in the `/storage/schroot/centos-6.5`):

    $ mkdir -p /storage/schroot/centos-6.5
    $ rpm --root=/storage/schroot/centos-6.5 --rbuilddb

This will add an empty database to `/storage/schroot/onixs.centos-6.5/var/lib/rpm`.
To enforce `yum` work we have to install some configuration files for it.
The easiest way to get them for your desired distro is to install `centos-release` package.

    $ wget http://mirror.centos.org/centos/6.5/os/x86_64/Packages/centos-release-6-5.el6.centos.11.1.x86_64.rpm
     <skip details>
    $ rpm --root=/storage/schroot/onixs.centos-6.5 --nodeps -i centos-release-6-5.el6.centos.11.1.x86_64.rpm
    warning: Generating 12 missing index(es), please wait...
    warning: centos-release-6-5.el6.centos.11.1.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
    $ rm centos-release-6-5.el6.centos.11.1.x86_64.rpm

This will install repository configuration files to `${chroot}/etc/yum.repos.d/`. Edit that files
according your needs: personally I want some extra repositories to be enabled in `CentOS-Base.repo`.
Also `yum` wants to check RPM package signature, so here is few options to deal with:

* make a symbolic link:

        $ ln -s /storage/schroot/centos-6.5/etc/pki /etc/pki

* … or disable signature checking (temporarily) in `CentOS-Base.repo` config file

<div class="alert alert-info" markdown="1">
#### TODO

Better to write a `ebuild` w/ CentOS keyring keys, just like Debian/Ubuntu has (I've seen it in some repo).
</div>

Now we can do the next step -- install a minimal base system:

    $ yum --installroot=/storage/schroot/centos-6.5 update
    base                                                  | 3.7 kB  00:00:00
    centosplus                                            | 3.4 kB  00:00:00
    contrib                                               | 2.9 kB  00:00:00
    extras                                                | 3.4 kB  00:00:00
    updates                                               | 3.4 kB  00:00:00
    (1/5): contrib/primary_db                             | 1.2 kB  00:00:00
    (2/5): base/primary_db                                | 4.4 MB  00:00:01
    (3/5): centosplus/primary_db                          | 1.5 MB  00:00:01
    (4/5): extras/primary_db                              |  19 kB  00:00:03
    (5/5): updates/primary_db                             | 3.2 MB  00:00:04
    Resolving Dependencies
    --> Running transaction check
    ---> Package centos-release.x86_64 0:6-5.el6.centos.11.1 will be updated
    ---> Package centos-release.x86_64 0:6-5.el6.centos.11.2 will be an update
    --> Finished Dependency Resolution

    Dependencies Resolved

    =======================================================================
     Package              Arch     Version              Repository    Size
    =======================================================================
    Updating:
     centos-release       x86_64   6-5.el6.centos.11.2  updates       20 k

    Transaction Summary
    =======================================================================
    Upgrade  1 Package

    Total download size: 20 k
    Is this ok [y/N]: y
    Downloading packages:
    warning: /storage/schroot/onixs.centos-6.5/var/cache/yum/updates/packages/centos-release-6-5.el6.centos.11.2.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
    Public key for centos-release-6-5.el6.centos.11.2.x86_64.rpm is not installed
    centos-release-6-5.el6.centos.11.2.x86_64.rpm                                                                                   |  20 kB  00:00:00
    Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    Importing GPG key 0xC105B9DE:
     Userid     : "CentOS-6 Key (CentOS 6 Official Signing Key) <centos-6-key@centos.org>"
     Fingerprint: c1da c52d 1664 e8a4 386d ba43 0946 fca2 c105 b9de
     Package    : centos-release-6-5.el6.centos.11.1.x86_64 (installed)
     From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    Is this ok [y/N]: y
    Running transaction check
    Running transaction test
    Transaction test succeeded
    Running transaction
      Updating   : centos-release-6-5.el6.centos.11.2.x86_64            1/2
      Cleanup    : centos-release-6-5.el6.centos.11.1.x86_64            2/2
      Verifying  : centos-release-6-5.el6.centos.11.2.x86_64            1/2
      Verifying  : centos-release-6-5.el6.centos.11.1.x86_64            2/2

    Updated:
      centos-release.x86_64 0:6-5.el6.centos.11.2

    Complete!

    $ yum --installroot=/storage/schroot/centos-6.5 install -y yum
    <a lot of messages skipped>

The last command brings me 91 packages… Unfortunately Gentoo has next versions of `rpm` and `yum`,
so "native" CentOS can't understand the database. So, the rest installs better to do from inside a chrooted
environment, but clean (yep, again) the RPMs database.

    zaufi@gentop〉~〉 build-chroot
    1) chroot:centos-6.5
    #? 1
    [schroot] password for builder:
    -bash: mesg: command not found
    (onixs.centos-6.5)builder@gentop:~$ su
    Password:
    bash-4.1# rm -rf /var/lib/rpm
    bash-4.1# rpm --rebuilddb
    bash-4.1# rpm --nodeps -i /var/cache/yum/updates/packages/centos-release-6-5.el6.centos.11.2.x86_64.rpm
    warning: /var/cache/yum/updates/packages/centos-release-6-5.el6.centos.11.2.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
    warning: /etc/yum.repos.d/CentOS-Base.repo saved as /etc/yum.repos.d/CentOS-Base.repo.rpmorig
    bash-4.1# yum install yum
    <skip details>

Now chroot is ready to use ;-)

See Also
--------

* Add [RPMForge repository](http://wiki.centos.org/AdditionalResources/Repositories/RPMForge)
* [HOWTO: Make a chroot'ed Debian/Ubuntu]({% post_url 2012-09-02-howto-make-a-debian-chroot %}) -- read this
  to get details about `schroot` configuration files. This chroot uses almost the same config, except
  one line in `/etc/schroot/rhbuilder/fstab` file:

        /storage/soft/centos/yum     /var/cache/yum    none    rw,bind    0 0

