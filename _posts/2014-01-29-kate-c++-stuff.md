---
layout: post
title: "Kate: Python plugins for C++ developers"
description: ""
category: programming
tags: [kde, kate, C++, howto]
---
{% include JB/setup %}


Nowadays [kate](http://kate-editor.org) has few things implemented as Python plugins and dedicated
to C++ programmers. Some of them (like working w/ comments) can be used in Python, CMake, Bash or JS
programming as well. To start to play w/ them one have to enable the following plugins:

<img src="/assets/images/kate.cpp/pate-plugins.png" class="img-rounded img-responsive" title="Plugins to be enabled" />

When take a look to Pâté menu:
<img src="/assets/images/kate.cpp/pate-menu.png" class="img-rounded img-responsive" title="Actions became available" />


Few words about working w/ comments
-----------------------------------


* _Inline comment_ means that it placed on a line w/ code. Just like on a screenshot above. To add it
    to a current line use `Alt-D` cursor will move to 60th (default) position and `// ` will be added.
    To transform it into a doxygen comment (`///<`) just press one `/` (this part work w/ 
    [my C++ not-quite-indenter™]({% post_url 2013-11-29-kate-cppstyle-indenter %}) ;-)
    To configure default position use _Commentar plugin_ config page (take a look to the pages
    available at the first screenshot). Pressing on a selection will add (if possible) inline
    comments to every selected line.

* To move inline comment to a line above use `Meta+Left` and `Meta+Right` to move it back
    <img src="/assets/images/kate.cpp/inline-comment.gif" class="img-rounded img-responsive" title="Inline Comment" />

* The next example show how to _Comment Block with `#if 0`_, _Toggle `#if0`/`#if1`_, and _Remove `#if 0`_ part
    <img src="/assets/images/kate.cpp/if0-block.gif" class="img-rounded img-responsive" title="Block Comment" />

* _Transform Doxygen Comments_ (between `/** ... */` and `///` forms) and _Shink/Expand Comment Paragraph_
    <img src="/assets/images/kate.cpp/dox.gif" class="img-rounded img-responsive" title="Shrink/Expand Paragraph" />



Boost MPL-like parameters fold/unfold
-------------------------------------

To format template or function parameters in a boost (MPL) style, move cursor inside of parenthesis or
angle brackets and use _Boost Like Format Params_ (`Meta+F`) action from the _Pâté_ menu. 
Every time this action will expand one (the current) nesting level. To unfold parameters use the reverse
_Unformat_ (`Meta+Shift+F`) action.
<img src="/assets/images/kate.cpp/boost-format.gif" class="img-rounded img-responsive" title="Boost Format" />



Using <code>expand</code> plugin
--------------------------------

Since a long time ago <del>in a galaxy far, far away</del> Pâté has the `expand` plugin. The (initial) idea
behind was quite simple: user writes a function which returns a string to be inserted into a document
(by _Expand_ action) instead of a corresponding function name under cursor. Expand functions may have
parameters. Running in a context of emebbed Python, that functions may have an access to kate's API.
The other benefit: this is still a (full functional) Python with a lot of third party libraries.


### Trivial example

The "Hello World" demo function as simple as
{% highlight python %}
def hi():
    return 'Hello from expand plugin'
{% endhighlight %}

To make this expand available put this function into a file named after a MIME type, where `/` replaced with `_`. 
For example `~/.kde4/share/apps/kate/pate/expand/text_html.expand` to make it available in a HTML documents
or `text_x-python.expand` for Python source code.

Then open a document with a corresponding MIME type, add `hi` text and press `Ctrl+E` (default) shortcut --
`hi` will be replaced w/ `Hello from expand plugin` text. 

Lets rewrite the code a little and introduce an optional parameter:

{% highlight python %}
def hi(name=None):
    return 'Hello {}'.format(name if name is not None else 'anonymous')
{% endhighlight %}

To pass a parameter type `hi(John)` + `Ctrl+E`. Note, that expansion function may have variable and named parameters,
just like any other ordinal python function.


### Level 2: Templating templates

Kate has a builtin [template engine](http://api.kde.org/4.x-api/applications-apidocs/kate/ktexteditor/html/classKTextEditor_1_1TemplateInterface.html#a03b041eaf934e8ddddfe922bda64d4aa).
Snippets and [file templates](http://docs.kde.org/stable/en/applications/kate/kate-application-plugin-filetemplate.html) 
use it. Nowadays `expand` can use it as well :) Yeah, the string `'Hello {}'` can be considered as a level 1 
template -- a `name` parameter will be substituted into. The _level 2_ is to allow to edit it after expansion, 
just like the snippets plugin (yeah, it is actually a plugin ;-). All that you need is to return a string with 
_editable field_, i.e. smth like `'Hello ${name:John}'` -- here is an editable field `name` with a "predefined" 
value `John` which is an actual parameter to expand function `hi` ;-) Also we ought to tell to expand plugin
that a result string must be inserted into a document via `KTexeEditor::TemplateInterface`, so we use
a Python decorator `postprocess` from the module `expand`:

{% highlight python %}{% raw %}
import expand

@expand.postprocess
def hi(name=None):
    return 'Hello ${{name:{}}}'.format(name if name is not None else 'anonymous')
{% endraw %}{% endhighlight %}

This way a partly editable piece of code can be inserted! Yeah, right, just like that snippets from a plugin...
But wait! _"Why the heck we've just reinvent the wheel?"_ -- one may ask ;-) The benefit that we can return
__any template we want!__ -- i.e. not just a static text with few editable holes (which is a "classic" snippet is)!
What to return can be (trans)formed according expansion function parameters... and this is a __real__ advantage
over snippets! :-) Still do not believe? Here is a _real world example_: lets write an expansion function to
insert a structure definitions to C++ code (it doesn't use editable fields for simplicity):

{% highlight python %}{% raw %}
_BRIEF_DOC_TPL = '''\
/**
 * \\brief Struct \\c {0}
 */'''

_TEMPLATE_PARAMS_TPL = '''
template <{1}>'''

_STRUCT_BODY_TPL = '''
struct {0}
{{
}};
'''

def st(name, *templateParams):
    params = [name]
    if len(templateParams):
        template = _BRIEF_DOC_TPL + _TEMPLATE_PARAMS_TPL + _STRUCT_BODY_TPL
        params.append('typename ' + ', typename '.join(templateParams))
    else:
        template = _BRIEF_DOC_TPL + _STRUCT_BODY_TPL
    return template.format(*params)
{% endraw %}{% endhighlight %}

This pretty simple (and quite short) function can do something, that snippets __just can't!__

{% highlight c++ %}
// st(test) expands to

/**
 * \brief Struct \c test
 */
struct test
{
};


// but st(templated_test, T, U) expands to

/**
 * \brief Struct \c templated_test
 */
template <typename T, typename U>
struct templated_test
{
};
{% endhighlight %}

The only mandatory parameter (the first one) is a structure name to insert. The optional rest, if present,
turned into a template parameters… That is how `st` function was look like before KDE CS 4.13.

But, sooner after trying to use builtin Python's `format` function with "template"
strings intensively, I've realized that it is time to use a real _template engine_ ;-)


### Level 3: Metatemplates

Template engines are widely used in web programming to split _business logic_ from a _representation_.
One of the famous one for Python is [jinja](http://jinja.pocoo.org/) (yeah, it is not an all-inclusive-framework… 
just a template engine ;-) can be used with `expand` plugin.

Lets rewrite structure expansion function using jinja templates:

{% highlight python %}
import expand

@expand.jinja('struct.cpp.tpl')
@expand.postprocess
def st(name=None, *templateParams):
    return {'name': name, 'template_params': templateParams}
{% endhighlight %}

Yeah, that's it! :-) Just returns a Python _dictionary_ with arbitrary keys. That keys will be used in a template
file later, so how to name them is up to you. The representation (template) file (`struct.cpp.tpl`), 
mentioned in a `@expand.jinja` decorator call, must be placed in a `${KDEDIR}/apps/kate/pate/expand/templates/` dir,
"near" the corresponding `.expand` plugin file.
.

{% raw %}
    /**
     * \brief Struct /*< name | editable >*/
     */
    /*% if template_params %*/
    template <
        /*% for tp in template_params %*/
            /*% if not loop.first %*/, /*% endif %*/typename /*< tp | editable >*/
        /*% endfor %*/
      >
    /*% endif %*/
    struct /*< name | editable(active=True) >*/
    {
        ${cursor}
    };
    //# kate: hl C++; remove-trailing-spaces false;
{% endraw %}

where,

* `/*<` and `>*/` open/close sequence to substitute a [variable](http://jinja.pocoo.org/docs/templates/#variables) --
  yeah, one of the keys in a dictionary, returned by an expand function
* `/*%` and `%*/` open/close sequence for [jijna control tags](http://jinja.pocoo.org/docs/templates/#list-of-control-structures)
  (like `if`, `endif`, `for`, `endfor`)
* `//#` is a _short_ or _single-line_ form of a comment inside a template file
* here is a one custom [filter](http://jinja.pocoo.org/docs/templates/#filters) used: `editable` -- to transform a value of a
  variable (key) into an editable `KTE::TemplateInterface` field after snippet gents inserted. Also note that
  structure name present in a comment and in a `struct` declaration line, but only the second one will be "editable",
  the other "copies" are just _mirrors_ in terms of `KTE::TemplateInterface`
* everything else, i.e. a text out of open/close sequences, is just a text to be rendered and inserted into a document
* both open and close sequences are not occasional: because it is template for C++ code, it would be nice to have some highlighting
  and chosen sequences are subset of plain C comments ;-) Sure, it can be configured for other languages…

<p>
<del>To reduce syntactic noise</del> For demonstration purposes I've removed <a href="http://jinja.pocoo.org/docs/templates/#whitespace-control">whitespace control</a> 
characters from jinja tags.
</p>

Here are couple few decorators defined in `expand` module to use by expand functions: `@expand.description` and `@expand.details`.
Both are accept a string to be displayed in a completion popup. The first one is a short description. The second one
is a _details_ (or usage) text accessible by pressing `Alt` key for selected completion item. Because most of completions are
really short names, completions will not appear in automatic mode, so to see them, one have to call that popup manually.
Here are few expansion functions available for C++ code "ot of the box", and the most complicated (but powerful if compare 
with trivial snippets) is `cl`. It is capable to generate template or non-template class, possible with zero (default) 
or N-arity constructor, maybe with defaulted or deleted copy and/or move constructor and/or assign operator, maybe with 
virtual destructor, where a class name and possible template and/or constructor parameters are (post)editable fields.
All parameters, also as an expand function name itself, are really short names (trying to be memorable as well) to
reduce typing effort.

<img src="/assets/images/kate.cpp/cl-completion.png" class="img-rounded img-responsive" title="Expand Functions Completions" />

One example can tell more than 100 words… lets generate a class with:
* template parameter `Iter` followed by `T0`, `T1`, `T2`, `T3`
* default constructor
* constructor accepting two parameters
* disabled copy constructor
* defaulted move constructor
* and virtual destructor
{% highlight c++ %}
// cl(some,@c,c2,-cc,@mc,vd,t,Iter,T0..3)
// result is:

/**
 * \brief Class \c some
 */
template <typename Iter, typename T0, typename T1, typename T2, typename T3>
class some
{
public:
    /// Default constructor
    some() = default;
    /// Make an instance using given parameters
    some(param0, param1)
    {
    }
    /// Copy constructor
    some(const some&) = delete;
    /// Copy assign
    some& operator=(const some&) = delete;
    /// Move constructor
    some(some&&) = default;
    /// Move assign
    some& operator=(some&&) = default;
    /// Destroy an instance of some
    virtual ~some()
    {
    }
};

{% endhighlight %}

I wouldn't tell about other functions -- it is easy to read descriptions/sources and play with them.
But I have one more final <del>damn cool</del> feature to tell about ;-)


### Level 80: God Mode

… to be continue …
