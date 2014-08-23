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

![Color Tools](/assets/images/color-tools/enable-color-tools.png){:.img-rounded.img-responsive}

The plugin provides a _Palette_ tool-view which contains all colors found in the current document.
For example here is the palette for [`http://cdn.kde.org/css/bootstrap.css`](http://cdn.kde.org/css/bootstrap.css):

![CSS file from dot.kde.org](/assets/images/color-tools/color-tools-1.png){:.img-rounded.img-responsive}

When a text with some `#hex` color gets selected, a tooltip with that color will appear.
Clicking on a color cell in the tool-view moves a cursor to the location, where that color has defined.
This plugin add a new action to the context menu of a document:

![Insert color action](/assets/images/color-tools/color-tools-2.png){:.img-rounded.img-responsive}

This action show a "completion" popup with a color-chooser widget containing all used colors,
so it is easy to choose some, using a keyboard or mouse.


![Color insert completion popup](/assets/images/color-tools/color-tools-3.png){:.img-rounded.img-responsive}

<div class="alert alert-warning" markdown="1">
#### Note

**Screenshots 1 and 3 are made of KDE SC 4.13**

</div>


<div class="alert alert-info" markdown="1">
#### TODO

* Support for other color syntaxes like `rgb(R,G,B)`, `rgba(R,G,B,A)`, `hsl(...)`, & etc.

</div>
