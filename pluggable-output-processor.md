---
layout: page
title: "Pluggable Output Processor"
description: "Abour Pluggable Output Processor"
---
{% include JB/setup %}


Usage Examples
==============

CMake
-----

CMake module can recognize (and colorize) various detection <del>SPAM</del> info during test phase.
Also it will reduce a little lines count to be printed by one trick: if a previous line (remembered
internally after print) is a subset of the current one, just move one line above and print over it...

Before:
![My helpful screenshot](assets/images/cmake-before.png "Title")
After:
![My helpful screenshot](assets/images/cmake-after.png "Title")


GNU make
--------

This module can colorize _error_ and _service messages_ from `make` (like _entering/leaving_ directory).
Also it can recognize some "information" messages from `cmake` and `gcc` command line (when `make VERBOSE=1` 
used to build) -- i.e. it works for cmake-based projects. 

Before:
![My helpful screenshot](assets/images/make-before.png "Title")
After:
![My helpful screenshot](assets/images/make-after.png "Title")


GNU gcc
--------

This module is capable to colorize C++ code snippets (best viewed w/ 256 color terminals ;-) and reformat/simplify
some loooong error messages...

Before:
![My helpful screenshot](assets/images/gcc-before.png "Title")
After:
![My helpful screenshot](assets/images/gcc-after.png "Title")


How to extend
-------------

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
