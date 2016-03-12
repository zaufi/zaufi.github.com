---
layout: post
title: "Realign o'keys in paludis output"
description: "How to fix [ok] placement w/ paludis"
category: gentoo
tags: [gentoo, paludis, problem]
---
{% include JB/setup %}


I've started to use [paludis](http://paludis.exherbo.org) around 4-5 years ago.
Unfortunately it has a bug (I'm in doubt that it is actually a feature :-) --
all `[ ok ]` it prints are placed at 80th position... or so, I don't remember exactly,
so sometimes they are placed over a text (like _"Applying patch blah-blah-very-long-name..."_)
and in any case on a wide terminal window it look strange...

So here is a mine fix (hack actually) for that, I've done a long time ago for dynamic 
detection of a terminal size:

```bash
# Put this to the end of your /etc/paludis/bashrc
save_COLUMNS=${COLUMNS}
COLUMNS=$(stty size 2>/dev/null | cut -d ' ' -f2)
test -z "${COLUMNS}" && COLUMNS=${save_COLUMNS}
unset save_COLUMNS
PALUDIS_ENDCOL=$'\e[A\e['$(( ${COLUMNS:-80} - 7 ))'G'
```

Have fun! :-)
