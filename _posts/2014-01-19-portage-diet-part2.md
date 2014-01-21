---
layout: post
title: "Part 2: Put the portage tree on a diet ;-)"
description: "How to remove unused files from the tree"
category: gentoo
tags: [gentoo]
---
{% include JB/setup %}


After getting a feedback about not quite complete clean and a
[generator script]({% post_url 2013-08-02-portage-diet %}), which needs to be (not forgetten to) 
rerun all the time after `.skel` file has modified, I've (re)read `man rsync` 
again paying attention to filtering rules description.

I was given a hint, that the script actually is not required. Everything can be done using only `rsync`
filtering rules. The key feature is (I missed that, first time reading it ;-)

<blockquote><p class="text-info">
    If pattern starts w/ a slash it is matched to the root of transfer, otherwise it is matched 
    against the end of current item path...
</p></blockquote>

Hence is it possible to remove a whole category or a particular directory (package) whithing some
category and its cached files from `metadata/md5-cache/` with a single rule.

    # rule to ignore a whole category
    - cat/***

    # rule to accept only one package from a category
    + cat/some-package**
    - cat/**

One more implication to consider when you writing filetering rules: **rules order is important**. 
You have to specify _include_ rules, then more generic _exclude_. It stops matching if first match 
(include or exclude) was found. So if you want to remove everything, but `amd64/` from `/profiles/arch/`

    + /profiles/arch/amd64/**
    - /profiles/arch**

Then, if you want to "clean" unused features of selected arch (like `x32` or `no-multilib`), you need
to add that rules **above**:

    - /profiles/arch/amd64/no-multilib/**
    - /profiles/arch/amd64/x32/**
    + /profiles/arch/amd64/**
    - /profiles/arch**

Unfortunately that approach do not scale well ;-( Particularly because the way `rsync` visits subdirectories:
**shortest paths matched first**. I.e. you can't write smth like this:

    + /profiles/default/linux/amd64/**
    - /profiles/default/**

because before visit nested `amd64/` the `/profiles/default/linux/` dir will be visited, but the only 
rule it matches is the second one: _kill all inside of `/profiles/default/`_, and the first rule will never match!
So to do a pedantic cleanup of `/profiles` one have to write a lot of rules to explicitly enumerate things
deep inside of profiles related dirs...

But there is a other side of a coin: the way `rsync` visits and applies rules actually can help to describe desired
cleanups in a **shorter functional-like** way! :-)

Lets describe what we want to have/keep... Personally I use _latest gentoo linux release (it is 13.0), native 64-bit
amd64 with multilib and KDE as a desktop_. Everything else I don't need! First of all we have to take care
about all files, [that makes up a profile](http://dev.gentoo.org/~ulm/pms/5/pms.html#x1-460005.2), **anywhere**
inside the `/profiles/` dir:

    + /profiles/**/eapi
    + /profiles/**/make*
    + /profiles/**/parent
    + /profiles/**/package*
    + /profiles/**/profile*
    + /profiles/**/use*
    + /profiles/**/ChangeLog

**Note** the last rule is not part of the profile, but will keep latest `ChangeLog` files, killing the
outdated others (`ChangeLog-2007` for example). Observing nested dirs inside a `/profiles` one may notice
that desired things to keep are placed in a dirs named after architecture or features, so to include them
(prevent deletion) it would be enough the following bunch of rules:

    + /profiles/**/amd64
    + /profiles/**/linux
    + /profiles/**/13.0
    + /profiles/**/64bit-native
    + /profiles/**/multilib
    + /profiles/**/multilib/lib32
    + /profiles/**/desktop
    + /profiles/**/kde

And the final part: **just kill everything else!!** That `/profile/arch/base/` dir must be kept due referenced
from particular arch dir.

    + /profiles/arch/base**
    - /profiles/arch/**
    - /profiles/default/**
    - /profiles/embedded/***
    - /profiles/features/**
    - /profiles/hardened/***
    - /profiles/prefix/**
    - /profiles/releases/**
    - /profiles/targets/**
    - /profiles/uclibc/***


### Measure the effect

I have downloaded and unpack a portage snapshot for 2014-Jan-14 to play w/ into `/storage/tmp/p`. 
Wrote a simple script in it to get a filtered copy of `portage/` -- just a shortcut for `rsync`
w/ a bunch of options:

{% highlight bash %}
#!/bin/sh

rsync --exclude-from=./gentoo-portage-exclude.list \
      --delete-excluded \
      --delete-missing-args \
      --recursive \
      --delete-after \
      --verbose \
      portage/ portage.stripped/
{% endhighlight %}

Now lets count files+dirs before/after and total directory size I've got with my
[final version of filtering rules](https://github.com/zaufi/paludis-config/blob/master/repositories/gentoo-portage-exclude.list):

    zaufi@gentop〉/storage/tmp/p〉 find portage | wc -l
    178370

    zaufi@gentop〉/storage/tmp/p〉 find portage.stripped | wc -l
    119408

    zaufi@gentop〉/storage/tmp/p〉 du -hs portage
    739M    portage

    zaufi@gentop〉/storage/tmp/p〉 du -hs portage.stripped
    503M    portage.stripped


<div class="alert alert-success">
<h4>Results</h4>
<ul>
    <li><strong>236M</strong> freed</li>
    <li><strong>58,962</strong> directory entries are eliminated</li>
</ul>
</div>

