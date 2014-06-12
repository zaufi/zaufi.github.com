---
layout: post
title: "Tuning autotoolized packages via <code>EXTRA_ECONF</code>"
description: ""
category: gentoo
tags: [gentoo, autotools, ebuild, optimization]
---
{% include JB/setup %}


Ebuilds for autotoolized packages has ability to pass extra options to underlaid
`./configure` scripts. It is supposed (I guess) that most of the time it doesn't needed to end users.
It is recommended to use when you want to add some per-package configuration option(s)
without necessity to hack a particular ebuild.

But actually there is at least one candidate to be added for **all**
(autotoolized) packages: `--disable-dependency-tracking`. It is responsible to add special options to
compiler CLI (like `-MD`, `-MP`, `-MF`), so latter, along w/ output of object file, will generate a
`make` formatted dependencies for particular source. That is quite useful when develop software, but
completely useless (and waste time, CPU, disk I/O and space) for one time compilation (like emerging
a package).

Also, to reduce <del>spam</del> amount of compilation details in autotoolized packages,
one may add `--enable-silent-rules` to turn this

    libtool: compile:  gcc -std=gnu99 -DHAVE_CONFIG_H -I. -I../../../../libX11-1.6.2/modules/om/generic
    -I../../../src -I../../../include/X11 -I../../../../libX11-1.6.2/include -I../../../../libX11-1.6.2/
    include/X11 -I../../../include -I../../../include/X11 -I../../../../libX11-1.6.2/src/xcms -I../../..
    /../libX11-1.6.2/src/xkb -I../../../../libX11-1.6.2/src/xlibi18n -I../../../../libX11-1.6.2/src -D_B
    SD_SOURCE -D_BSD_SOURCE -DHAS_FCHOWN -DHAS_STICKY_DIR_BIT -DMALLOC_0_RETURNS_NULL -Wall -Wpointer-ar
    ith -Wmissing-declarations -Wformat=2 -Wstrict-prototypes -Wmissing-prototypes -Wnested-externs -Wba
    d-function-cast -Wold-style-definition -Wdeclaration-after-statement -Wunused -Wuninitialized -Wshad
    ow -Wmissing-noreturn -Wmissing-format-attribute -Wredundant-decls -Werror=implicit -Werror=nonnull
    -Werror=init-self -Werror=main -Werror=missing-braces -Werror=sequence-point -Werror=return-type -We
    rror=trigraphs -Werror=array-bounds -Werror=write-strings -Werror=address -Werror=int-to-pointer-cas
    t -Werror=pointer-to-int-cast -fno-strict-aliasing -g -O2 -MT omTextExt.lo -MD -MP -MF .deps/omTextE
    xt.Tpo -c ../../../../libX11-1.6.2/modules/om/generic/omTextExt.c  -fPIC -DPIC -o .libs/omTextExt.o
    libtool: compile:  gcc -std=gnu99 -DHAVE_CONFIG_H -I. -I../../../../libX11-1.6.2/modules/om/generic
    -I../../../src -I../../../include/X11 -I../../../../libX11-1.6.2/include -I../../../../libX11-1.6.2/
    include/X11 -I../../../include -I../../../include/X11 -I../../../../libX11-1.6.2/src/xcms -I../../..
    /../libX11-1.6.2/src/xkb -I../../../../libX11-1.6.2/src/xlibi18n -I../../../../libX11-1.6.2/src -D_B
    SD_SOURCE -D_BSD_SOURCE -DHAS_FCHOWN -DHAS_STICKY_DIR_BIT -DMALLOC_0_RETURNS_NULL -Wall -Wpointer-ar
    ith -Wmissing-declarations -Wformat=2 -Wstrict-prototypes -Wmissing-prototypes -Wnested-externs -Wba
    d-function-cast -Wold-style-definition -Wdeclaration-after-statement -Wunused -Wuninitialized -Wshad
    ow -Wmissing-noreturn -Wmissing-format-attribute -Wredundant-decls -Werror=implicit -Werror=nonnull
    -Werror=init-self -Werror=main -Werror=missing-braces -Werror=sequence-point -Werror=return-type -We
    rror=trigraphs -Werror=array-bounds -Werror=write-strings -Werror=address -Werror=int-to-pointer-cas
    t -Werror=pointer-to-int-cast -fno-strict-aliasing -g -O2 -MT omTextExt.lo -MD -MP -MF .deps/omTextE
    xt.Tpo -c ../../../../libX11-1.6.2/modules/om/generic/omTextExt.c -o omTextExt.o >/dev/null 2>&1
    mv -f .deps/omTextExt.Tpo .deps/omTextExt.Plo
    /bin/sh ../../../libtool  --tag=CC   --mode=compile gcc -std=gnu99 -DHAVE_CONFIG_H -I. -I../../../..
    /libX11-1.6.2/modules/om/generic -I../../../src -I../../../include/X11  -I../../../../libX11-1.6.2/i
    nclude -I../../../../libX11-1.6.2/include/X11 -I../../../include -I../../../include/X11 -I../../../.
    ./libX11-1.6.2/src/xcms -I../../../../libX11-1.6.2/src/xkb -I../../../../libX11-1.6.2/src/xlibi18n -
    I../../../../libX11-1.6.2/src -D_BSD_SOURCE  -D_BSD_SOURCE -DHAS_FCHOWN -DHAS_STICKY_DIR_BIT    -DMA
    LLOC_0_RETURNS_NULL -Wall -Wpointer-arith -Wmissing-declarations -Wformat=2 -Wstrict-prototypes -Wmi
    ssing-prototypes -Wnested-externs -Wbad-function-cast -Wold-style-definition -Wdeclaration-after-sta
    tement -Wunused -Wuninitialized -Wshadow -Wmissing-noreturn -Wmissing-format-attribute -Wredundant-d
    ecls -Werror=implicit -Werror=nonnull -Werror=init-self -Werror=main -Werror=missing-braces -Werror=
    sequence-point -Werror=return-type -Werror=trigraphs -Werror=array-bounds -Werror=write-strings -Wer
    ror=address -Werror=int-to-pointer-cast -Werror=pointer-to-int-cast -fno-strict-aliasing -g -O2 -MT
    omTextPer.lo -MD -MP -MF .deps/omTextPer.Tpo -c -o omTextPer.lo ../../../../libX11-1.6.2/modules/om/
    generic/omTextPer.c

into this

    make[3]: Entering directory '/storage/tmp/paludis/x11-libs-libX11-1.6.2/work/libX11-1.6.2-amd64/modu
    les/om/generic'
    CC       omDefault.lo
    CC       omGeneric.lo
    CC       omImText.lo
    CC       omText.lo
    CC       omTextEsc.lo
    CC       omTextExt.lo
    CC       omTextPer.lo
    CC       omXChar.lo
    CCLD     libxomGeneric.la
    make[3]: Leaving directory '/storage/tmp/paludis/x11-libs-libX11-1.6.2/work/libX11-1.6.2-amd64/modu
    les/om/generic'

Here is a related part of `/etc/paludis/bashrc`

{% highlight bash %}
EXTRA_ECONF="--disable-dependency-tracking --enable-silent-rules"
CMAKE_VERBOSE=OFF
{% endhighlight %}

<div class="alert alert-info" markdown="1">
#### Note

The last option (I have to mention) has the same effect (reduce spam) for cmake based packages which
are by default (in eclass) too verbose.
</div>
