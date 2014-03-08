---
layout: post
title: "Kate: Color tools"
description: "Helper plugin to edit colors"
category: programming
tags: [kde, kate, css, howto]
---
{% include JB/setup %}

Since KDE SC 4.12 `kate` has a Python plugin which can help to edit colors in various files
like CSS, kate syntax, HTML, QML… First of all, one ought to enable that plugin in the
_Python plugins manager_ dialog.

<img src="/assets/images/color-tools/enable-color-tools.png" 
    class="img-rounded img-responsive" 
    title="Color Tools" 
  />

The plugin provides a _Palette_ tool-view which contains all colors found in the current document.
For example here is the palette for [`http://cdn.kde.org/css/bootstrap.css`](http://cdn.kde.org/css/bootstrap.css):

<img src="/assets/images/color-tools/color-tools-1.png" 
    class="img-rounded img-responsive" 
    title="CSS file from dot.kde.org" 
  />

When a text with some `#hex` color gets selected, a tooltip with that color will appear.
Clicking on a color cell in the tool-view moves a cursor to the location, where that color has defined.
This plugin add a new action to the context menu of a document:

<img src="/assets/images/color-tools/color-tools-2.png" 
    class="img-rounded img-responsive" 
    title="Insert color action" 
  />

This action show a "completion" popup with a color-chooser widget containing all used colors,
so it is easy to choose some, using a keyboard or mouse.


<img src="/assets/images/color-tools/color-tools-3.png" 
    class="img-rounded img-responsive" 
    title="Color insert completion popup" 
  />

