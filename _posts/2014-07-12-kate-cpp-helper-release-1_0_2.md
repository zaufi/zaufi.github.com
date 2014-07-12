---
layout: post
title: "Kate C++ Helper plugin release 1.0.2"
description: "Kate C++ Helper plugin release 1.0.2"
category: C++
tags: [kde,kate,C++]
---
{% include JB/setup %}

Here is a changes list in this, mostly cosmetic, [release](http://kde-apps.org/content/show.php/?content=148606):

* Add import/export completions sanitizer rules -- it uses KDE configs API and doesn't intended to edit by humans.
  The primary goal was to be able to store and exchange rules among users/hosts.
* Provide sample, but quite useful, sanitizer rules as a pre installed file w/ my rules exported.
  See [mini howto page](/kate-cpp-helper-plugin.html) about what is _completions sanitizer_ and how to use it.
* Keep current (session specific) _include paths set_ selected after store it.
* Add action to step back (i.e. return to a previous location) after lookup declaration/definition of some symbol.
* Set default shortcuts for all lookup actions same as ctags plugin has. Yeah, you don't need that plugin anymore ;-)
* Eliminate compile error w/ gcc 4.9. Fix [issue #18](https://github.com/zaufi/kate-cpp-helper-plugin/issues/18)
* Few cleanups in UI.
* A bunch of improvements in build system (not interested to end-users ;-)
