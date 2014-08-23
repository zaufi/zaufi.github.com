---
layout: post
title: "Kate: Alternative (pure) C++14 ready syntax highlighting"
description: "Alternative (pure) C++14 ready syntax highlighting"
category: programming
tags: [kde, kate, C++]
---
{% include JB/setup %}

Motivation
==========

Since kate developers decided to use my C++/Qt4 syntax as default for C++ highlighting,
I (re)started to maintain my ["fork"](http://kde-files.org/content/show.php?content=90660)
of alternative pure C++ syntax as a separate project. The main reason for that: I want to
have **pure** C++ syntax as default, because most of code I do, do not use Qt framework
and I don't want to see irrelevant highlighting, when use `boost::signals2` or POSIX sockets.

Also I have plans to add a separate C++/Qt5 syntax for Qt5 based project (and mark some Qt4
stuff in it as deprecated).


Changes against the "official" syntax
-------------------------------------

I've aready done a bunch of bug fixes and refactorings. Also support for upcoming C++14 features
has been added. Partial change log:

* support for quote digits separator
* support for user defined literals defined by The Standard for strings, complex numbers and
  `std::chrono` constants
* highlighting for header files `#incldue` defined by The Standard to make them visually different
  than ordinal `#include`, to give a hint that you write it's name correctly ;)
* support for user defined literals for floating point numbers (C++11 actually)
* _Aligned Comments_ now is a separate attribute and by default has the same style as _Comment_.
  To reduce a number of attributes it was a _Regon Marker_, so a lot of users was wondered
  why some comments (at some positions) was highlighted as a _Region Marker_.


About _Aligned Comments_
------------------------

Most of C++ code I've seen (and work with) uses 4 spaces to indent. I always try to align my comments
to TAB-stops that aliquot to (my favorite) indent size -- i.e. 0, 4, 8, …, 60.
But sometimes mass replacements (like `sed`) may lead to _misalign_ some comments which, sometimes, is not
easy to notice. It is why I've added that feature to my highlighter -- to make properly aligned comments
(little) visually different than others.

![C++ highlighting example](/assets/images/kate-cpp-syntax.png){:.img-rounded.img-responsive}

The 60th position is the last (in my code), where an _inline-comment_ may appear -- it is why aligned
comments are detected only _before_ this position… To properly align _inline-comments_ I'm using
my ["Not quite indenter"]({% post_url 2013-11-29-kate-cppstyle-indenter %})™ and/or a
[Pâtè plugins dedicated to C++ progrmmers]({% post_url 2014-02-13-kate-c++-stuff %}).


See Also
--------

* [Download](http://kde-files.org/content/show.php?content=90660) alternative pure C++ syntax
* Git [repository](https://github.com/zaufi/kate-stuff/tree/master/syntax) on GitHub
