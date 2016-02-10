---
layout: page
title: "My paludis hooks and addons"
description: "Tips and tricks"
---
{% include JB/setup %}


Short Intro
-----------

In my [paludis-hooks](https://github.com/zaufi/paludis-hooks) repo I'm trying to accumulate everything
I've done <del>to survive w/</del> for [paludis](http://paludis.exherbo.org) -- The Other Package Manager™.
Yep, initially there was only [hooks](http://paludis.exherbo.org/configuration/hooks.html), but later I've
added some other helpful things, I'm using for awhile... so probably the repo should be renamed.

Personally I like hooks, cuz that facility, out of the box, turns `paludis` into a really flexible package manager
and thanks to them, feature like _autopatch_ works much more better (and reliably) than the
[same feature](https://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?part=3&chap=6#doc_chap6) of
`emerge` became available long time after.

To install all that stuff, here is a
[ebuild](https://github.com/zaufi/zaufi-overlay/blob/master/sys-apps/paludis-hooks/paludis-hooks-scm.ebuild)
in my overlay. And yes, that `scm` is the only version nowadays ;-) cuz I'm permanently doing some improvements
w/o any release plan.

**NOTE**: To use it w/ Python 3 as a default interpreter make sure you have an
[appropriate](/gentoo/2013/12/11/paludis-python3/) paludis version installed.


Things in Details
=================

Autopatch Hook
--------------

One of my first hooks inspired by an already dead repo of paludis' hooks, I've found soon after migrated to
paludis. That was one from a bunch of reasons to forget about `emerge` -- a really easy way to apply my patches
for broken packages w/o "writing" a my own ebuild and put it to my repo.

The idea is simple and straightforward: at any desired stage of ebuild processing apply some _user provided patches_
placed into a some special place (configurable via `/etc/paludis/hooks/configs/auto-patch.conf`).

To apply a patch one have to put it into a directory under `${PATCH_DIR}/stage/category/pkgname-with-ver/`
or to `${PATCH_DIR}/stage/category/pkgname/`, so you don't need to create a new directory (or symlink a previous 
`pkg-spec-with-ver`) when package version has bumped. Where the `stage` is one of the following: 
`ebuild_compile_post`, `ebuild_compile_pre`, `ebuild_configure_post`, `ebuild_configure_pre`, `ebuild_install_pre` 
or `ebuild_unpack_post`.

Recently I've killed all out of date patches from my local vault and made it
[public](https://github.com/zaufi/paludis-autopatches). For example w/o a patch `cmake` won't colorize its
output even if color explicitly requested when running w/ captured output. So to allow my
[Pluggable Output Processor](/pluggable-output-processor.html) works w/o suppressing colors of CMake I have
`/var/db/paludis/autopatches/ebuild_unpack_post/dev-utils/cmake/cmake-3.0.0-do-not-check-isatty.patch`.


Filesystem Hook
---------------

This plugin can be used to make some manipulations in a package's image, right after `make install`
and before it will be actually merged into the system. Particularly I use it to make a _permanent_
symlinks to a documentation for some packages I use in my work or to prevent undesired files to be installed
(which is far better than ugly `INSTALL_MASK`).

Every action to apply described in terms of XML items of the configuration file
`/etc/paludis/hooks/configs/filesystem-manager.conf`, where the root element is `commands`.
A commands container consists of `package` elements which must have at least the `spec` attribute of a package
to apply actions for. The other attributes are `stop`, with boolean `true`/`false` value to disable/enable
further processing, if a given rule matched, and the `descr` attribute with a human readable action description.
The package's `spec` attribute specify the order how actions are matched and applied:

<div class="row">
    <div class="col-md-4">
        <div class="table-responsive">
            <table class="table">
                <thead>
                    <tr><th>priority</th> <th>spec</th></tr>
                </thead>
                <tr><td>16</td> <td>cat/package-ver-rv:slot::repo</td></tr>
                <tr><td>15</td> <td>cat/package-ver-rv:slot</td></tr>
                <tr><td>14</td> <td>cat/package-ver-rv::repo</td></tr>
                <tr><td>13</td> <td>cat/package-ver-rv</td></tr>
                <tr><td>12</td> <td>cat/package:slot::repo</td></tr>
                <tr><td>11</td> <td>cat/package:slot</td></tr>
                <tr><td>10</td> <td>cat/package::repo</td></tr>
                <tr><td>9</td>  <td>cat/package</td></tr>
                <tr><td /> <td/></tr>
            </table>
        </div>
    </div>
    <div class="col-md-4">
        <div class="table-responsive">
            <table class="table">
                <thead>
                    <tr><th>priority</th> <th>spec</th></tr>
                </thead>
                <tr><td>8</td> <td>package-ver-rv:slot::repo</td></tr>
                <tr><td>7</td> <td>package-ver-rv:slot</td></tr>
                <tr><td>6</td> <td>package-ver-rv::repo</td></tr>
                <tr><td>5</td> <td>package-ver-rv</td></tr>
                <tr><td>4</td> <td>package:slot::repo</td></tr>
                <tr><td>3</td> <td>package:slot</td></tr>
                <tr><td>2</td> <td>package::repo</td></tr>
                <tr><td>1</td> <td>cat/*</td></tr>
                <tr><td>0</td> <td>*/*</td></tr>
            </table>
        </div>
    </div>
</div>

There are few children tags possible under the `package` element:

* `symlink` -- used to create a symlink and has attributes:
    * `cd` -- change to this directory before making a symlink
    * `src` -- source for the symlink
    * `dst` -- destination of the symlink
* `rm` -- used to remove something from the image, so it will not be installed at all.
    * `dst` -- what to remove
    * `reverse` -- optional attribute to specify that command should remove **everything** except
      selected target(s)
* `if` -- used to check some (limited nowadays) constraints and execute nested actions if latter is `true`
    * `use` -- the only mandatory attribute nowadays to check if given `USE` flag is turned on for matched package
    * `negate` -- boolean attribute to negate result of a use check

Paths in a rule may refer to shell variables like `P`, `PN`, `PF`, `PV`, `PVR`, as well as others,
declared by [PMS](http://wiki.gentoo.org/wiki/Project:PMS).
Note, that all absolute paths are refer to the package's image directory and not to the "real" filesystem.

### Examples

#### Make permanent symlinks to documentation

This was an initial motivation to write this hook: I use the `firefox` to view a documentation to various packages.
Documentation for `=dev-lib/boost-1.54`, for example, will be installed to the `/usr/share/doc/boost-1.54`.
Firefox will remember that location and next time shows me hints (in a popup) with the most visited pages, when I
type something in a location bar. Unfortunately after upgrade to `=dev-libs/boost-1.55-r1` path to documentation
obviously will be changed and that hints will not work without fixing. The second problem with that:
I have to remember an exact package version with a ebuild's revision number and should type it in a location bar a
long time until firefox realized that visits count of some `1.55-r1` page is greater than same, but with `1.54`
in the path, and will offer it instead. Except `boost`, I've got about 10 more packages w/ `USE=doc` flag enabled
and I don't want to remember all versions and revisions I currently have in my system... and bookmarks in
firefox can't help much in this case!

The solution is to make a "permanent" symbolic link which will stay the same for any version installed.
Moreover it can be a little shorter :-) So instead of `/usr/share/doc/boost-1.55.0-r1/html/index.html` I have
`/usr/share/doc/boost/index.html` as a hint in a location bar and don't care about particular version installed.

To achieve that I've added a simple rule to my config:
{% highlight xml %}
<package spec="dev-libs/boost">
    <symlink cd="/usr/share/doc" src="${PF}/html" dst="${PN}" />
</package>
{% endhighlight %}
So, this rule will change current directory to `/usr/share/doc` inside an image directory and create
a symbolic link `boost` pointing to `boost-1.55-r1/html/`.


#### Remove unused translations/locales

I have `*/* -nls` and `LUNGUAS: -*` in my `/etc/paludis/use.conf`, but some packages just don't have that USE flag,
but install localizations anyway (yep, cuz ebuild authors just lazy ppl... most of the time).
So `app-admin/localpurge` was "invented" to cleanup unused locales (all of them in my case). But `localepurge`
will remove `*.mo` files after install, so when a package gets uninstalled, some files will be marked as _gone_.
One simple rule will do the job better:
{% highlight xml %}
<package spec="*/*" descr="locale-cleaner">
    <rm cd="/usr/share/locale/" dst="*/LC_MESSAGES/*.mo" />
</package>
{% endhighlight %}
Because manipulations (deleting `*.mo` files) will be done **before** merge, all that files even
won't be counted by a package manager. And I'm not telling about that you don't need to run any tool periodically
(or via `cron`) -- all your packages will be already clean w/o any manual actions required :)

Translations is a part of the "problem": some packages (like `alsa-utils`) want to install translated manual pages
as well. To remove them (everything except English) one may use the following rule:
{% highlight xml %}
<package spec="*/*" descr="man-pages-cleaner">
    <rm cd="/usr/share/man/" dst="man{0p,1,1p,2,3,3p,4,5,6,7,8}" reverse="true" />
</package>
{% endhighlight %}
Note the attribute `reverse` tells to the hook that everything except specified items (directories actually)
should be removed.

Other usage examples can be found
[here](https://github.com/zaufi/paludis-config/blob/master/hooks/configs/filesystem-manager.conf).
They are mostly targeted to remove unused/redundant/useless files installed by various packages to
the `/usr/share/doc` and some other places.


#### Even more extreme rule

Recently I've added one more element: `if` -- to check if a matched package has some desired `USE` flag turned on.
Nowadays I have the following __extreme__ rule in my config:

{% highlight xml %}
<package spec="*/*" descr="USE=-doc remover">
    <if use="doc" negate="true">
        <rm cd="/usr/share" dst="doc" />
    </if>
</package>
{% endhighlight %}

<div class="alert alert-danger" markdown="1">
#### Attention

This rule will prevent to install any files to the `/usr/share/doc/` from all packages,
which do not have `USE=doc` enabled!
</div>

Unfortunately some packages, I want docs for, do not have that `USE` at all, so to stop this rule triggering,
I've got a bunch of _stoppers_ like this:

{% highlight xml %}
<package spec="clang-docs" stop="true" />
<package spec="python-docs" stop="true" />
<package spec="valgrind" stop="true" />
{% endhighlight %}


Working Directory in a RAM
--------------------------

Some users have a paludis woking directory mounted to a `tmpfs`. The big problem w/ this solution:
some packages are too fat to be build in a RAM. Then to build them one have to `umount` selected location
temporarily to build that packages. `workdir-tmpfs` hook is designed to solve that "problem".

This hook record a disk space usage while a package gets build and the next time will try to move
the working directory to the `/dev/shm` (providing a symbolic link to the original location) if there
is enough space. Disk usage stats will be recorded only if a package builds successful.
To move a working directory to a RAM disk it should have enough space: previously collected value plus
some reserved space. Basically amount of bytes to reserve is a 10% of the stored value.

Default configuration file location is `/etc/paludis/hooks/configs/workdir-tmpfs.conf`.
The hook's configuration has the following default values:

{% highlight bash %}
MIN_RESERVED_SPACE=$(( 1024 * 1024 * 10 ))
MAX_RESERVED_SPACE=$(( 1024 * 1024 * 100 ))
DISK_USAGE_STATS_CACHE="/var/cache/paludis/disk_usage_stats.cache"
{% endhighlight %}

`MIN_RESERVED_SPACE` is a value to be used as an amount of a reserved space if 10% of a stored disk usage
counter is less than it. The `MAX_RESERVED_SPACE` is an upper bound of that 10% -- i.e. max reserved space
can not exceed it. Defaults are 10M and 100M correspondingly.

<div class="alert alert-info" markdown="1">
#### Note

If some package fails to build, its working directory will be moved back to its original location
from the RAM disk (i.e. `/dev/shm`)...
</div>

Manage Build Environments
-------------------------

This feature is not a hook actually, but some add-on to be used from the `/etc/paludis/bashrc`.
It allows to manage compiler flags (as well as any other environments variables) on a per package basis,
like it is possible in the portage via the [`package.env` file](https://wiki.gentoo.org/wiki//etc/portage/env).

To use it, one have to add the following to the `/etc/paludis/bashrc` somewhere after the `CFLAGS`
and/or other definitions.

{% highlight bash %}
[ -e /usr/libexec/paludis-hooks/setup_pkg_env.bash ] && . /usr/libexec/paludis-hooks/setup_pkg_env.bash
{% endhighlight %}

Now you can edit the `/etc/paludis/package_env.conf` to specify how environment should be changed to compile
any particular package. For example, I have somethig like 
[this](https://github.com/zaufi/paludis-config/blob/hardware/notebook/MSI-GP60-2PE-Leopard/package_env.conf):

    #
    # My per package environment settings
    #

    sys-devel/binutils binutils-extras

    dev-lang/python extra-optimize
    dev-db/sqlite extra-optimize
    sys-apps/paludis extra-optimize

    sys-libs/db use-bfd-linker

    app-emulation/virtualbox no-gc-sections use-bfd-linker
    media-libs/alsa-lib no-gc-sections
    sys-devel/glibc no-gc-sections
    x11-libs/cairo no-gc-sections

    net-libs/webkit-gtk no-debug
    www-client/firefox no-debug


Every line has a package spec and a space separated list of environment modifiers. They would be applied in the same order
as specified in the config. Every modifier actually is a file from the `/etc/paludis/env.conf.d/`. 
In that file (an ordinal bash script actually to be sourced if a package name matched) defined some actions to modify 
`CFLAGS`, `LDFLAGS` or anything else you want. For example in 
[`/etc/paludis/bashrc`](https://github.com/zaufi/paludis-config/blob/hardware/notebook/MSI-GP60-2PE-Leopard/bashrc) 
I have the following definition for `LDFLAGS`:

{% highlight bash %}
LDFLAGS="-Wl,-O1 -Wl,--sort-common -Wl,--as-needed -Wl,--enable-new-dtags -Wl,--gc-sections -Wl,--hash-style=gnu"
{% endhighlight %}

Unfortunately, not all packages can link with `--gc-sections` option. For example `app-emulation/virtualbox`.
It is why this package have `no-gc-sections` environment in my `package_env.conf`. You may use all functions provided by 
[`flag-o-matic.eclass](https://devmanual.gentoo.org/eclass-reference/flag-o-matic.eclass/index.html) to manupulate
compiler/linker flags! Here is mine `/etc/paludis/env.conf.d/no-gc-sections.conf` for example:

{% highlight bash %}
einfo "Remove --gc-sections from linker flags"
filter-ldflags -Wl,--gc-sections
{% endhighlight %}

Yes, you can use logging functions as well ;-) The last command will remove undesired option from default
("global") `LDFLAGS`. Unfortunately `app-emulation/virtualbox` also fails to build if default linker is `ld.gold`.
The second environment (`use-bfd-linker`) tells to use `ld.bfd` to link that package:

{% highlight bash %}
if [ "bfd" != "$(readlink -f `which ld` | sed 's,.*\.\(bfd$\),\1,')" ]; then
    einfo "Use ld.bfd linker"
    append-ldflags -Wl,-fuse-ld=bfd
fi
{% endhighlight %}

Also there is a buch of other helper functions provided -- you may check 
[my paludis configuration](https://github.com/zaufi/paludis-config/tree/master/env.conf.d)
for inspiration. Being an _ordinal bash script_ one may use any other programs, `sed` for example, to modify environment 
and/or build flags for particular package.


New `cave` Subcommand
---------------------

This package also add a "new" subcommand `cave print-ebuild-path <spec>` which just prints a full path to the
ebuild according a given `spec`.

    zaufi@gentop〉~〉 cave print-ebuild-path firefox
    /usr/portage/www-client/firefox/firefox-26.0.ebuild

    zaufi@gentop〉~〉 cave print-ebuild-path '<www-client/firefox-26.0'
    /usr/portage/www-client/firefox/firefox-24.2.0.ebuild

    zaufi@gentop〉~〉 cave print-ebuild-path paludis-hooks
    /var/db/paludis/repositories/zaufi-overlay/sys-apps/paludis-hooks/paludis-hooks-scm.ebuild

This command (mostly) introduced for further scripts.


Short Aliases and Some Other Helpers
------------------------------------

I use the following short aliases for the `cave` subcommands:

{% highlight bash %}
alias cs='cave search --index ${CAVE_SEARCH_INDEX}'
alias cm='cave manage-search-index --create ${CAVE_SEARCH_INDEX}'
alias cc='cave contents'
alias cr='cave resolve'
alias crz='cave resolve ${CAVE_RESUME_FILE_OPT}'
alias cw='cave show'
alias co='cave owner'
alias cu='cave uninstall'
alias cy='cave sync'
alias cz='cave resume -Cs ${CAVE_RESUME_FILE_OPT}'
alias world-up='cave resolve ${CAVE_RESUME_FILE_OPT} -c -km -Km -Cs -P "*/*" -Si -Rn world'
alias system-up='cave resolve ${CAVE_RESUME_FILE_OPT} -c -km -Km -Cs -P "*/*" -Si -Rn system'
{% endhighlight %}

Variables `CAVE_SEARCH_INDEX` and `CAVE_RESUME_FILE_OPT` are defined in `/etc/env.d/90cave` file.
Also here some trick used to reintroduce bash completions for all that commands, so even if alias is used
completions are still available.

Note that `-km` and `-Km` [options are used]({% post_url 2013-12-21-gpotw-5 %}). So when gentoo team adds
some changes to ebuild w/o version bump, that options will add a target to rebuild. To show differences
between ebuilds of installed package and just changed (and added as a target to rebuild on `world-up`
"command") one may use `pkg_meta_diff <pkg-name>` command.

To show differences between installed and the next best available ebuilds use `pkg_ebuild_diff <pkg-name>`.

Also here is a couple of functions to use: `ebuild_for` -- to get a path of the ebuild according a
given spec and `show_ebuild_for` to `less` it...


TODO
====

* Add more commands! Like `*zip` smth...
* Add ability to find target objects (files, dirs, whatever) by introducing smth
  like `find` item and iterate over results applying some other actions (`ln`, `rm`, & etc...)
* Implement FSM commands as **real** plugins... need to think about how to update (merge) DTD then.
* add some `USE` flags to `paludis-hooks` live
  [ebuild](https://github.com/zaufi/zaufi-overlay/blob/master/sys-apps/paludis-hooks/paludis-hooks-scm.ebuild)
  to be able to choose what components to install...
* Process `symlink` elements as they are followed in a configuration file. Nowadays this order is **undefined**.
* Allow to "override" `src` attribute in `rm` and `symlink/item` tags:

{% highlight xml %}
<package spec="cat/pkg">
    <symlink cd="/usr/share/bash-completion/completions" src="some-bash-completion.sh">
        <item dst="uno" />
        <item dst="dos" />
        <!-- Override -->
        <item src="other-bash-completion.sh" dst="tres" />
    </symlink>
</package>
{% endhighlight %}


See Also
========

* [repository](https://github.com/zaufi/paludis-config) w/ my paludis configuration. Note, that it has various branches for my hardware.
