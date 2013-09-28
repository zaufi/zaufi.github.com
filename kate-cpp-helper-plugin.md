---
layout: page
title: "Kate C++ Helper Plugin"
description: "Kate C++ Helper Plugin project page"
---
{% include JB/setup %}


Mini HowTo
==========

This page describe actions to configure the plugin for your project.

Installation
------------

Unpack (or <a data-original-title="$ git clone https://github.com/zaufi/kate-cpp-helper-plugin.git" 
href="#" data-toggle="tooltip" data-placement="top" title="">clone</a>) sources to some workng directory, then execute the following commands from it:

    $ cd <plugin-sources-dir>
    $ mkdir build && cd build
    $ cmake -DNO_DOXY_DOCS=ON -DBUILD_TESTING=OFF -DCMAKE_INSTALL_PREFIX=$(kde4-config --localprefix) ..
    $ make
    $ make install



Configuration
-------------

To enable the plugin for your current session use _Settings -> Configure Kate... -> Plugins_:

![Enable the C++ Helper Plugin](assets/images/cpphelper/enable.png "Enable the C++ Helper Plugin")

Now use plugins' configuration page to ajdust some settings. First of all make sure the _System Paths List_
have a proper directories list used by your compiler (`gcc` by default). Initially, the plugin try to detect
known compilers (gcc and clang) from your path. If current list is not suitable for you, clear it and 
choose appropriate compiler from a list below the page and press _Add to List_.

<div class="alert alert-info">
Support for compilers not in the <code>PATH</code> in my TODO list (for example manually built fresh or crosscompiler gcc).
The <em>Other</em> (disabled now) option has reserved for this purpose.
</div>

![System-wide Settings](assets/images/cpphelper/system.png "System-wide Settings")

Add anything else that as you think belongs to all `kate` sessions.

The next page is a _Session Paths List_. Here is a `#include` directories list for your **current session**.
You can store this list into a configuration file for future quick load using _Favorite and Stored Sets_.
By default the plugin provides one predefined set `Qt4` with directories reported by Qt detection script
at plugin compilation time.

Typical dirs you have to add here is the following:
* root of your project's source/build tree and/or any other directories inside your project
* paths to libraries, used in the project, as reported by corresponding configuration tools

The plugin also try to suggest some directories based on opened documents' paths.

<div class="alert alert-info">
Personally I use only two <code>#include</code> directories in all my projects: source tree root and build root.
And every source file contains a full path relative to one of above when <code>#include</code> something.
Particularly this technique helps to simplify configuration management when you have to add a bunch of 
<code>-I</code> options in your build system configuration files and track inter-components dependencies
all the time you've added a new <code>#include</code> directive in you sources... so finally you've got
a particular compiler command line options the half of your terminal window long :)
</div>

![Session-wide Settings](assets/images/cpphelper/session.png "Session-wide Settings")

The next page allows you to add more compiler options, such as:

* defines found by your build system or required by third party libraries used, so code completer
  will see (and suggest) appropriate declarations
* do not forget to add `-xc++` if you have a C++ project. Also consider to add `-std=c++11` and
  any other options which could affect internal compiler's predefined macros

![CLang Settings](assets/images/cpphelper/clang-settings.png "CLang Settings")

The plugin is capable to use a _Precompiled Headers_ file. If you have it for your project
is it recommended to specify it in _Session PCH header to compile_ option. Typically this header
consist of `#include` directives for most used header files in your sources. For `cmake` based projects
I have a [module](https://github.com/mutanabbi/chewy-cmake-rep/blob/master/UsePCHFile.cmake) which helps
to make it by command `make update-pch-header`. To use it just add `UsePCHFile.cmake` and 
`PreparePCHHeader.cmake.in` files to your cmake modules directory of your project and in top `CMakeLists.txt`
call it like this:
{% highlight cmake %}
include(UsePCHFile)
use_pch_file(
    PCH_FILE ${CMAKE_BINARY_DIR}/my-project-pch.hh
    EXCLUDE_DIRS cmake docs
  )
{% endhighlight %}
Then you may add this file to _CLang Settings_ page and plugin will build it and use while code completion.

<div class="alert alert-success">
The mentioned CMake module actually is a part of a <a href="https://github.com/mutanabbi/chewy-cmake-rep" target="_blank">
collection of reusable modules</a> under control of <a href="https://github.com/mutanabbi/chewy" target="_blank">
<code>chewy</code> utility</a>. To install it using <code>chewy</code> you can use:
<pre>
$ chewy install https://raw.github.com/mutanabbi/chewy-cmake-rep/master/UsePCHFile.cmake
</pre>
any required additional files will be installed automatically.
</div>

The next configuration page of the plugin allows you to fine tune completion results.

<div class="alert alert-error">
<em>Automatic code completion</em> option nowadays is a quite stupid: it tries a completion 
after user enters <em>'.'</em> or <em>'->'</em> in a source code. But more important that it can annoy on
a heavy project (with <a href="http://boost.org" target="_blank">Boost</a> library for example).
</div>

![Code Completion Settings](assets/images/cpphelper/completion-settings.png "Code Completion Settings")

The most important thing on this page is a set of regular expressions that can be used to filter
completion results. For example, Boost Preprocessor library define a lot of internal symbols which are
not a part of public API (same for some other libraries from Boost). Also unlikely you want to see
various `#incldue` guards... Using completion results sanitizer you may enter a regex like below
to filter them out:

    BOOST_(PP_[A-Z_]+_(\d+|[A-Z])|.*_HPP(_INCLUDED)?$|[A-Z_]+_(AUX|DETAIL)_)

Leaving a replacement part empty instruct code completer to remove matched completions. Another practical
example of usage sanitizer is to beautify some verbose names and transform them into a shorter. For example
completer may contain items with `std::basic_string<char>`, which is well known as `std::string`, so to make
a particular completion result slightly shorter I use the regex like on a screen shot...

<div class="tabbable">
    <ul class="nav nav-tabs">
        <li class="active"><a data-toggle="tab" href="#no-sanitizer">Without sanitizer</a></li>
        <li><a data-toggle="tab" href="#sanitizer">With sanitizer</a></li>
    </ul>
    <div class="tab-content">
        <div id="no-sanitizer" class="tab-pane active">
            <img src="assets/images/cpphelper/sanitizer-disabled.png" class="img-rounded" />
        </div>
        <div id="sanitizer" class="tab-pane">
            <img src="assets/images/cpphelper/sanitizer-enabled.png" class="img-rounded" />
        </div>
    </div>
</div>

The _Other Settings_ page contains options related to `#include` directives and code completer.
Plugin has action (default key is `Shift+F10`) to copy `#include` statement with current file
into a clipboard, so you may switch a document and just paste it. The first option in a 
_`#include` Helper Options_ group specify format of that preprocessor directive.

At the end of this group you may find a list of file wildcards to exclude from `#include` completion list.

![Other Settings](assets/images/cpphelper/other-settings.png "Other Settings")

Based on your system and session directories lists the plugin may highlight `#include` directives
in a source file with an error or warning background. The first one can occur then a file specified is
not found. That means you have to add some more paths to the system/session list. In case of multiple
files found (same name, but different paths) background turned to a warning. In that case it depends
on `-I` compiler options order which one will be actually processed.

To open a header file just move cursor on a line containing `#include` and press `F10` key.
If no valid directive recognized on a current line, a dialog with a list of currently included files will appear.
If you have no write permission to opened file (most likely files from `/usr/include`) read-only
mode will be applied automatically.

![Choose Header File](assets/images/cpphelper/choose-header-dialog.png "Choose Header File")


<div class="alert alert-info">
<h6>Open Header/Implementation Motivation</h6>
<p>
<code>kate</code> shipped with a plugin named <em>Open Header</em>, but sooner after I started to use it I've found
few cases when it can't helps me. Nowadays I have 2 "real life" <del>really annoying</del> examples when it fails:
</p>

<ul>
<li>
Often one may find a source tree with separate <code>${project}/src/</code> and <code>${project}/include</code> dirs.
So, when you are at some header from <code>include/</code> dir, that plugin obviously never will find 
your source file. And vise versa.
</li>

<li>
The second case: sometimes I have a really big class defined in a header file
(let it be <code>my_huge_application.hh</code>). It may consist of few dickers of methods each of which is
hundred lines or so. In that case I prefer to split implementation into several files and name them
after a base header like <code>my_huge_application_cmd_line.cc</code> (for everything related to command 
line parsing), <code>my_huge_application_networking.cc</code> (for everything related to network I/O), 
and so on. As you may guess <em>Open Header</em> plugin will fail to find a corresponding header for that
source files.
</li>
</ul>
</div>

Last settings group on _Other Settings_ page allow you to tune _Open Header/Implementation_ behaviour.
It is a good practice to have the very first `#include` directive in a `.cc`/`.cpp` file to be a corresponding
header. So, checking the first option in that group will add it to a list of candidates when you press `F11` key.

The second option (_Use wildcard source files search_) aimed to solve the second "real life" problem:
being in a header file and pressing `F11` it appends all `.cc`/`.cpp` files started w/ a current header name
to a list of candidates.

**That is all!** (briefly ;-)

<div class="alert alert-info">
One more thing I have mention: typing <code>#in</code> in C++ sources will automatically insert <code>#include</code>
text and starts file completer...
</div>

<img src="assets/images/cpphelper/include-completer.png" class="img-rounded" title="Include Files Completer" />


See Also
========

* [latest release](http://kde-apps.org/content/show.php/?content=148606)
* [sources repository](https://github.com/zaufi/kate-cpp-helper-plugin)
* [ChangeLog](https://github.com/zaufi/kate-cpp-helper-plugin/blob/master/Changes.md)
