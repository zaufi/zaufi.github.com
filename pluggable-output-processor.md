---
layout: page
title: "Pluggable Output Processor"
description: "Abour Pluggable Output Processor"
---
{% include JB/setup %}

What is This?
=============

_Pluggable Output Processor_ is an engine to wrap any executabe and capture its output through
a pluggable module to colorize it and/or (re)format.


Motivation
==========

_Pluggable Output Processor_ is my attempt to get rid of a bunch of various colorizers
from my system (like `colorgcc`, `colordiff`, `colorsvn`, ...) <del>and take everything under control</del> ;-).
Some of them are written on Perl (and I'm not a fun of it :-) and after few hacks I've made to improve
`colorgcc`, I realized that I don't want to <del>waste my time</del> learning Perl.

Yes, I know there is a lot of stuff like this, but I have few problems w/ it:

1. I'm not a Perl programmer (and don't want to be), but I feel constant intention to improve that
  programs in a different ways.
2. Some of that stuff is actually abandoned -- even if I can fix or improve smth, there is nobody I can send
  my patch to...
3. Some of that stuff is <del>too much</del> 
  <em><a data-original-title="Default tooltip" data-toggle="tooltip" title="" href="#">end-user oriented</a></em>
  <del>so inflexible</del> -- they can colorize
  almost everything via regular expressions and configuration files. The only problem I have w/ them:
  some things I'd like to colorize (or fix formatting) is **damn hard to express via regexes** ... 
  particularly because line-by-line processing implemented in that tools have no _state_... 
  (yep, it will be hard to code, and even harder to use having only configuratin files)


Features
========

* easy (to Python programmers ;-) to extend
* 256 color terminal support ;-) configuration files in addition to standard named colors
  may contain color definitions as `rgb(r,g,b)` or `gray(n)`
* colorizers for `make`, `cmake`, `gcc` out of the box (more plugins to come ;-)
* some modules are not just a stupid colorizers ;-) For example `gcc` can reformat text for
  better readability (really helps to understand template errors). Also `cmake` module can reduce
  amount of lines printed during test by collapsing test _intro_ message and _result_ into a single one.


Installation
============

Easy!

    $ tar -xzf outproc-X.Y.tar.gz
    $ cd outproc-X.Y
    $ sudo easy_install .

For Gentoo users there is a [live ebuild](https://github.com/zaufi/zaufi-overlay/blob/master/dev-util/pluggable-output-processor/pluggable-output-processor-scm.ebuild)
in my [repository](https://github.com/zaufi/zaufi-overlay/). Also (for Gentoo users again ;-)
`eselect` module from `contrib/` will be installed by the ebuild. Users of other distros have to
make a symlinks to required modules manually:

    $ ln -s /usr/bin/outproc /usr/lib/outproc/bin/<module-name>

and then make sure `/usr/lib/outproc/bin` placed __before__ `/usr/bin` (and anything else) in your 
user/system `PATH` environment.


Usage Examples
==============

CMake
-----

CMake module can recognize (and colorize) various detection <del>SPAM</del> info during test phase.
Also it will reduce a little lines count to be printed by one trick: if a previous line (remembered
internally after print) is a subset of the current one, just move one line above and print over it...

<div class="tabbable">
    <ul class="nav nav-tabs">
        <li class="active"><a data-toggle="tab" href="#cmake-before">Before</a></li>
        <li><a data-toggle="tab" href="#cmake-after">After</a></li>
    </ul>
    <div class="tab-content">
        <div id="cmake-before" class="tab-pane active">
            <img src="assets/images/cmake-before.png" class="img-rounded">
        </div>
        <div id="cmake-after" class="tab-pane">
            <img src="assets/images/cmake-after.png" class="img-rounded">
        </div>
    </div>
</div>


GNU make
--------

This module can colorize _error_ and _service messages_ from `make` (like _entering/leaving_ directory).
Also it can recognize some "information" messages from `cmake` and/or `gcc` command line (when `make VERBOSE=1` 
used to build) -- i.e. it works for cmake-based projects (my favorite build utility nowadays).

<div class="tabbable">
    <ul class="nav nav-tabs">
        <li class="active"><a data-toggle="tab" href="#make-before">Before</a></li>
        <li><a data-toggle="tab" href="#make-after">After</a></li>
    </ul>
    <div class="tab-content">
        <div id="make-before" class="tab-pane active">
            <img src="assets/images/make-before.png" class="img-rounded">
        </div>
        <div id="make-after" class="tab-pane">
            <img src="assets/images/make-after.png" class="img-rounded">
        </div>
    </div>
</div>


GNU gcc
--------

This module is capable to colorize errors, warnings, C++ code snippets (best viewed w/ 256 color terminals ;-)
and reformat/simplify some loooong error messages...

<div class="tabbable">
    <ul class="nav nav-tabs">
        <li class="active"><a data-toggle="tab" href="#gcc-before">Before</a></li>
        <li><a data-toggle="tab" href="#gcc-after">After</a></li>
    </ul>
    <div class="tab-content">
        <div id="gcc-before" class="tab-pane active">
            <img src="assets/images/gcc-before.png" class="img-rounded">
        </div>
        <div id="gcc-after" class="tab-pane">
            <img src="assets/images/gcc-after.png" class="img-rounded">
        </div>
    </div>
</div>


Configuration Details
=====================

// TBD


TODO
====

* continue to improve C++ tokenizer (few things can be better)
* unit tests for tokenizer
* test files w/ to cause various error messages from gcc (+ unit test for colorizer somehow)
* continue to improve `cmake` support (+ unit tests)
* turn `mount` output into a jumab readable look-n-feel
* colorize `df` depending on free space threshold
* colorize `diff` (easy! :-)
* `eselect` module to manage tools under control
* ask module is it want to handle a current command or we can do `execv` instead
* implement `STDIN` reader (pipe mode)


How to Extend
=============

To add a new module you have to name it after a tool you want to wrap (process output from) +
`.py` extension and put that module into `${python-site-packages-dir}/outproc/pp/` dir.
Then make a symlink from `/usr/bin/outproc` to `/usr/lib/outproc/bin/<wrapped-executable>` or 
use `eselect` in Gentoo. Being executed `outproc` will realize (from the symlink name) what binary
to execute and capture output from. All captured lines will be _piped_ through `your-module.Processor.handle_line()`.

A minimal plugin code will looks like this:
   {% highlight python %}

#!/usr/bin/env python
# -*- coding: utf-8 -*-

import outproc

class Processor(outproc.Processor):

    def __init__(self, config, binary):
        super().__init__(config, binary)
        # If you want some parameters from `/etc/outproc/<your-module>.conf
        # use `self.config` to get them...


    def handle_line(self, line):
        '''
            Handle one line from a wrapped executable
        '''
        # TODO transfrom the line and return it back...
        return line

   {% endhighlight %}
