---
layout: post
title: "New features of Kate C++ Helper plugin"
description: "Kate C++ Helper plugin release 1.0.3 and 1.0.4"
category: programming
tags: [kde,kate,C++]
---
{% include JB/setup %}

Changes since release 1.0.2:

* Simple preprocessor directive auto completion has been added. If there is no ambiguity it will
auto substitute an item. E.g. to type `#pragma` it would be enough to press `#` and `p`.
It is replacement for former `#in` "shortcut" to type `#include`…   

  ![Preprocessor and #include completer][p1]{: .img-rounded .img-responsive}

* Improved `#include` completion with better support for local (project) files. Some other
features (like _Open Header/Implementation_) was also improved to support local headers.   

* Introduced an action to toggle `#include` style -- i.e. between `<>` and `""` form.   

  ![Toggle #include style][p2]{: .img-rounded .img-responsive}   
  It is easy to transform a bunch of `#include`s using a shortcut (default <kbd>Alt+#</kbd>).   

  ![Toggle #include style action][p3]{: .img-rounded .img-responsive}

* And as usual, also here is a bunch of refactorings to make a code better (thanks to C++14 ;-).

[p1]: /assets/images/cpphelper/preprocessor-and-include-completion.gif "Preprocessor and #include completer"
[p2]: /assets/images/cpphelper/toggle-include.png "Toggle #include style"
[p3]: /assets/images/cpphelper/toggle-include.gif "Toggle #include style action"


<div class="alert alert-success" markdown="1">
#### Attention

Since then version 1.0.3, it is required C++14 capable compiler to build this plugin -- 
i.e. GCC >= 4.9 or Clang >= 3.5.
</div>
