---
layout: post
title: "Kate: Python plugins for C++ developers"
description: ""
category: programming
tags: [kde, kate, C++, howto]
---
{% include JB/setup %}


Nowadays [kate](http://kate-editor.org) has few things implemented as Python plugins and dedicated
to C++ programmers. Some of them (like working w/ comments) can be used in Python, CMake, Bash or JS
programming as well. To start to play w/ them one have to enable the following plugins:

<img src="/assets/images/kate.cpp/pate-plugins.png" class="img-rounded img-responsive" title="Plugins to be enabled" />

When take a look to Pâté menu:
<img src="/assets/images/kate.cpp/pate-menu.png" class="img-rounded img-responsive" title="Actions became available" />

### Few words about working w/ comments

* _Inline comment_ means that it placed on a line w/ code. Just like on a screenshot above. To add it
    to a current line use `Alt-D` cursor will move to 60th (default) position and `// ` will be added.
    To transform it into a doxygen comment (`///<`) just press one `/` (this part work w/ 
    [my C++ not-quite-indenter™]({% post_url 2013-11-29-kate-cppstyle-indenter %}) ;-)
    To configure default position use _Commentar plugin_ config page (take a look to the pages
    available at the first screenshot). Pressing on a selection will add (if possible) inline
    comments to every selected line.

* To move inline comment to a line above use `Meta+Left` and `Meta+Right` to move it back
    <img src="/assets/images/kate.cpp/inline-comment.gif" class="img-rounded img-responsive" title="Inline Comment" />

* The next example shows how to _Comment Block with `#if 0`_, _Toggle `#if0`/`#if1`_, and _Remove `#if 0`_ part
    <img src="/assets/images/kate.cpp/if0-block.gif" class="img-rounded img-responsive" title="Block Comment" />

* _Transform Doxygen Comments_ (between `/** ... */` and `///` forms) and _Shink/Expand Comment Paragraph_
    <img src="/assets/images/kate.cpp/dox.gif" class="img-rounded img-responsive" title="Shrink/Expand Paragraph" />

### Boost style format

To format template or function parameters in a boost (MPL) style, move cursor inside of parenthesis or
angle brackets and use _Boost Like Format Params_ (`Meta+F`) action from the _Pâté_ menu. 
Every time this action will expand one (the current) nesting level. To unfold parameters use the reverse
_Unformat_ (`Meta+Shift+F`) action.
<img src="/assets/images/kate.cpp/boost-format.gif" class="img-rounded img-responsive" title="Boost Format" />


### ... to be continue ...

* Using <code>expand</code> plugin...
