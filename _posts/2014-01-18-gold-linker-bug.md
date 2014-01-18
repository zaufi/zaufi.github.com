---
layout: post
title: "Critical bug w/ gold linker"
description: "Alert for all ld.gold users"
category: alert
tags: [problem, ld.gold]
---
{% include JB/setup %}

Recently I've found a critical [bug](https://sourceware.org/bugzilla/show_bug.cgi?id=16417) w/ `ld.gold`.
I'm lucky that it happened in a service paludis' utility `print_exports` used to form stored environment file
when a package gets build... do not even want to imagine what if the problem happened w/ `cave` executable...
or maybe some other most frequent used binary (like `bash`, `make`, `gcc` or `kdeinit`).

So nowadays I had <del>scared</del> finished to play w/ `ld.gold` and switched back to `ld.bfd` as a default linker...
at least while bug is not fixed...
