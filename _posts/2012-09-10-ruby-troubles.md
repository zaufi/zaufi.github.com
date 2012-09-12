---
layout: post
title: "[dev-lang/ruby] Do you know..."
description: "Interesting fact"
category: administration
tags: [ruby, problem]
---
{% include JB/setup %}

Recently I've faced w/ compile error on `=dev-ruby/json-1.7.5`
(a bug report is [here](https://bugs.gentoo.org/show_bug.cgi?id=434072)).
The final reason of this behaviour was surprising for me!

Also, nowadays cmake's (<=2.8.9) SWIG module has buggy support for Ruby :( It is why the recommented way
to copy `/usr/share/cmake/Modules/UseSWIG.cmake` to your modules directory and apply the following patch:
{% highlight diff %}

--- UseSWIG.cmake.orig  2012-08-20 15:17:19.000000000 +0400
+++ UseSWIG.cmake       2012-09-12 08:09:17.000000000 +0400
@@ -178,7 +178,7 @@
    "${swig_source_file_fullname}"
    MAIN_DEPENDENCY "${swig_source_file_fullname}"
    DEPENDS ${SWIG_MODULE_${name}_EXTRA_DEPS}
-    COMMENT "Swig source")
+    COMMENT "Swig source ${swig_source_file_fullname}")
SET_SOURCE_FILES_PROPERTIES("${swig_generated_file_fullname}" ${swig_extra_generated_files}
    PROPERTIES GENERATED 1)
SET(${outfiles} "${swig_generated_file_fullname}" ${swig_extra_generated_files})
@@ -238,6 +238,13 @@
    SET_TARGET_PROPERTIES(${SWIG_MODULE_${name}_REAL_NAME} PROPERTIES SUFFIX ".pyd")
    ENDIF(WIN32 AND NOT CYGWIN)
ENDIF ("${swig_lowercase_language}" STREQUAL "python")
+  #
+  # WARNING Remove `lib' prefix from Ruby modules!
+  # (standard cmake's script don't bother about this, need to report a bug)
+  IF ("${swig_lowercase_language}" STREQUAL "ruby")
+    # this is only needed for the python case where a _modulename.so is generated
+    SET_TARGET_PROPERTIES(${SWIG_MODULE_${name}_REAL_NAME} PROPERTIES PREFIX "")
+  ENDIF ("${swig_lowercase_language}" STREQUAL "ruby")
ENDMACRO(SWIG_ADD_MODULE)

#
{% endhighlight %}
