---
layout: post
title: "Bash: Hacking <code>PROMPT_DIRTRIM</code>med path"
description: "Replace ... with unicode triple underdot"
category: howto
tags: [bash, shell]
---
{% include JB/setup %}


`bash` (as some other shells) has one cute feature I used a long time:

<blockquote><em>
    <code>PROMPT_DIRTRIM</code>
    <br/>
    If set to a number greater than zero, the value is used as the number of trailing directory
    components  to  retain  when expanding the <code>\w</code> and <code>\W</code> prompt string escapes.
    Characters removed are replaced with an ellipsis.</em>
</blockquote>

Since ages `konsole` supports UTF-8 characters and they are used in my 
[smart bash prompt](http://cli-apps.org/content/show.php/smart-prompt?content=160498).
I've just wanted to check how easy it will be to replace the default `'...'` text w/ a single unicode 
character `'…'`. It took about 30 seconds to find out that unfortunately, `bash` have no option to set 
my own text instead of default. So I decided to make an 
["ugly hack"](https://github.com/zaufi/paludis-autopatches/blob/c268dcd66d8eeb991e17b79e1bfdd695134c6c69/ebuild_unpack_post/app-shells/bash-4.2_p45-r1/bash-4.2-unicode-triple-dot-trim-path.patch) ;-)

```diff
diff -ub -r bash-4.2.org/general.c bash-4.2/general.c
--- bash-4.2.org/general.c  2010-12-12 23:06:27.000000000 +0300
+++ bash-4.2/general.c  2014-02-02 21:25:00.000000000 +0400
@@ -761,9 +761,9 @@
   if (nlen <= 3)
     return name;
 
-  *nbeg++ = '.';
-  *nbeg++ = '.';
-  *nbeg++ = '.';
+  *nbeg++ = '\xe2';
+  *nbeg++ = '\x80';
+  *nbeg++ = '\xa6';
 
   nlen = nend - ntail;
   memcpy (nbeg, ntail, nlen);
```

`\xe2\xe80\xa6` is UTF-8 code for `…`

Thanks to my [autopatch hook](/my-paludis-hooks-and-addons.html) it is easy to build and install my hacked version
to see a result …

<img src="/assets/images/new-dirtrim-prompt.png" class="img-responsive" title=";try; Synopsis" />

The proper (not hackish) way should be:

0. get a value of `PROMPT_DIRTRIM_TEXT` if any, otherwise use default `'...'` string
0. send a patch back to upstream

... maybe later ;-)
