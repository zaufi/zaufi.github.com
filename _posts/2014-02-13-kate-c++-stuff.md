---
layout: post
title: "Kate: Python plugins for C++ developers"
description: ""
category: programming
tags: [kde, kate, C++, howto]
---
{% include JB/setup %}


Nowadays [kate](http://kate-editor.org) has a few things implemented as Python plugins that can be useful
to C++ programmers. Some of them (like working w/ comments) can be used wth Python, CMake, Bash or JS
code as well. To start playing w/ them one has to enable the following plugins:

![Plugins to be enabled](/assets/images/kate.cpp/pate-plugins.png){:.img-rounded.img-responsive}

Then take a look to Pâté menu:
![Actions became available](/assets/images/kate.cpp/pate-menu.png){:.img-rounded.img-responsive}


Few words about working with comments
=====================================


* _Inline comment_ means that it is placed on a line w/ code. Just like on the screenshot above. To add it
    to the current line use <kbd>Alt+D</kbd>. The cursor will move to 60th (default) position and `// ` will be added.
    To transform it into a doxygen comment (`///<`) just press one <kbd>/</kbd> (this part works w/ 
    [my C++ not-quite-indenter™]({% post_url 2013-11-29-kate-cppstyle-indenter %}) ;-)
    To configure the default position use the _Commentar plugin_ config page (take a look at the pages
    available at the first screenshot). Pressing on a selection will add (if possible) inline
    comments to every selected line.

* To move inline comment to a line above use <kbd>Meta+Left</kbd> and <kbd>Meta+Right</kbd> to move it back
    ![Inline Comment](/assets/images/kate.cpp/inline-comment.gif){:.img-rounded.img-responsive}

* The next example shows how to _Comment Block with `#if 0`_, _Toggle `#if0`/`#if1`_, and _Remove `#if 0`_ part
    ![Block Comment](/assets/images/kate.cpp/if0-block.gif){:.img-rounded.img-responsive}

* _Transform Doxygen Comments_ (between `/** ... */` and `///` forms) and _Shrink/Expand Comment Paragraph_
    ![Shrink/Expand Paragraph](/assets/images/kate.cpp/dox.gif){:.img-rounded.img-responsive}



Boost MPL-like parameters fold/unfold
=====================================

To format template or function parameters in the boost (MPL) style, move cursor inside the parenthesis or
angle brackets and use _Boost Like Format Params_ (<kbd>Meta+F</kbd>) action from the _Pâté_ menu. 
Every time invoked this action will expand the nesting level. To unfold parameters use the reverse
_Unformat_ (<kbd>Meta+Shift+F</kbd>) action.
![Boost Format](/assets/images/kate.cpp/boost-format.gif){:.img-rounded.img-responsive}



Using <code>expand</code> plugin
================================

Since long time ago <del>in a galaxy far, far away</del> Pâté had the `expand` plugin. The (initial) idea
behind it was quite simple: a user writes a function which returns a string to be inserted into the document
(by means of an _Expand_ action) instead of the corresponding function name under the cursor. Expand functions may have
parameters. Running in a context of embedded Python, these functions may access kate's API.
There is however another benefit - this still is a fully functional Python with a lot of third party libraries.


## Trivial example

The "Hello World" demo function is as simple as
{% highlight python %}
def hi():
    return 'Hello from expand plugin'
{% endhighlight %}

To make this expand available put this function into a file named after a MIME type, where `/` replaced with `_`. 
For example use `~/.kde4/share/apps/kate/pate/expand/text_html.expand` to make it available in a HTML documents
or `text_x-python.expand` for Python source code.

Then open a document with a corresponding MIME type, add `hi` text and press <kbd>Ctrl+E</kbd> (default) shortcut --
`hi` will be replaced w/ `Hello from expand plugin` text.

Lets rewrite the code a little and introduce an optional parameter:

{% highlight python %}
def hi(name=None):
    return 'Hello {}'.format(name if name is not None else 'anonymous')
{% endhighlight %}

To pass a parameter type `hi(John)` + <kbd>Ctrl+E</kbd>. Note, that the expand function may have positional and named parameters,
just like any other ordinary python function.


## Level 2: Templating templates

Kate has a builtin [template engine](http://api.kde.org/4.x-api/applications-apidocs/kate/ktexteditor/html/classKTextEditor_1_1TemplateInterface.html#a03b041eaf934e8ddddfe922bda64d4aa).
Snippets and [file templates](http://docs.kde.org/stable/en/applications/kate/kate-application-plugin-filetemplate.html) 
use it. Now, `expand` can use it as well :) Yeah, the string `'Hello {}'` can be considered as a level 1 
template -- a `name` parameter will be substituted into it. The _level 2_ is to allow to edit it after expansion,
just like with the snippets plugin (yeah, it is actually a plugin ;-). All you need is to return a string with an
_editable field_, i.e. smth like `'Hello ${name:John}'` -- here is an editable field `name` with a "predefined" 
value `John` which is an actual parameter to the expand function `hi` ;-) Also we ought to tell the expand plugin
that the result string must be inserted into the document via `KTexeEditor::TemplateInterface`, so we use
a Python decorator `postprocess` from the module `expand`:

{% highlight python %}{% raw %}
import expand

@expand.postprocess
def hi(name=None):
    return 'Hello ${{name:{}}}'.format(name if name is not None else 'anonymous')
{% endraw %}{% endhighlight %}

This way a partly editable piece of code can be inserted! Yeah, right, just like these snippets from a plugin...
But wait! _"Why the heck we've just reinvented the wheel?"_ -- one may ask ;-) The benefit is that we can return
__any template we want!__ -- i.e. not just a static text with a few editable holes (which is what a "classic" snippet is)!
What to return can be (trans)formed according to the expansion function parameters... and this is a __real__ advantage
over snippets! :-) Still do not believe? Here is a _real world example_: lets write an expansion function to
insert structure definitions to C++ code (it doesn't use editable fields for simplicity):

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

The only mandatory parameter (the first one) is a structure name to insert. The rest are optional, if present they are
turned into template parameters… That is how `st` function looked like before KDE SC 4.13.

But soon after trying to use the built-in Python's `format` function with "template"
strings intensively, I realized that it is time to use a real _template engine_ ;-)


## Level 3: Metatemplates

Template engines are widely used in web programming to split _business logic_ from a _representation_.
One of the famous for Python is [jinja](http://jinja.pocoo.org/). Yep, it is not an all-inclusive-framework… 
just a template engine ;-) and now we can use it with the `expand` plugin.

Lets rewrite the structure expansion function using jinja templates:

{% highlight python %}
import expand

@expand.jinja('struct.cpp.tpl')
@expand.postprocess
def st(name=None, *templateParams):
    return {'name': name, 'template_params': templateParams}
{% endhighlight %}

Yeah, that's it! :-) Just return a Python _dictionary_ with arbitrary keys. These keys will be used in a template
file later, so how to name them is up to you. The representation (template) file (`struct.cpp.tpl`), 
mentioned in the `@expand.jinja` decorator call, must be placed in a `${KDEDIR}/apps/kate/pate/expand/templates/` dir,
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

* `/*<` and `>*/`  open/close sequences to substitute a [variable](http://jinja.pocoo.org/docs/templates/#variables) --
  yeah, one of the keys in a dictionary, returned by the expand function
* `/*%` and `%*/` open/close sequence for [jijna control tags](http://jinja.pocoo.org/docs/templates/#list-of-control-structures)
  (like `if`, `endif`, `for`, `endfor`)
* `//#` is a _short_ or _single-line_ form of a comment inside a template file
* next, one custom [filter](http://jinja.pocoo.org/docs/templates/#filters) is used: `editable` -- to transform a value of a
  variable (key) into an editable `KTE::TemplateInterface` field after the snippet gets inserted. Also note that the
  structure name is present both in the comment and in the `struct` declaration line, but only the second one will be "editable",
  the other "copies" are just _mirrors_ in terms of `KTE::TemplateInterface`
* everything else, i.e. a text out of open/close sequences, is just a text to be rendered and inserted into a document
* both open and close sequences are chosen for a reason: since it is a template for the C++ code, it would be nice to have some highlighting
  and the chosen sequences are subsets of plain C comments ;-) Surely, it can be configured for other languages as well…

<p>
<del>To reduce syntactic noise</del> For demonstration purposes I've removed <a href="http://jinja.pocoo.org/docs/templates/#whitespace-control">whitespace control</a> 
characters from jinja tags.
</p>

Here are a couple more decorators defined in the `expand` module for use by expand functions: `@expand.description` and `@expand.details`.
Both accept a string to be displayed in a completion popup. The first one is a short description. The second one
is the _details_ (or usage) text accessible by pressing an <kbd>Alt</kbd> key for the selected completion item. Because most of the completions are
really short names, completions will not appear in automatic mode, so in order to see them, one has to bring up this popup manually.
There are a few expansion functions available for C++ code out-of-the-box. The most complicated one (but powerful compared
to trivial snippets) is `cl`. It is capable to generate template or non-template classes, possibly with a constructor with zero (default) 
or more argumentis, maybe with defaulted or deleted copy and/or move constructor and/or assign operator, maybe with a
virtual destructor, where the class name and the possible template and/or constructor parameters are (post)editable fields.
All parameters, and also the expand function's name should be really short to reduce the typing effort and be memorizable.

![Expand Functions Completions](/assets/images/kate.cpp/cl-completion.png){:.img-rounded.img-responsive}

One example is worth more than 1K words… lets generate a class with:

* template parameter `Iter` followed by `T0`, `T1`, `T2`, `T3`
* default constructor
* constructor accepting two parameters
* disabled copy constructor
* defaulted move constructor
* and virtual destructor
{% highlight c++ %}
// cl(some,@c,c2,-cc,@mc,vd,t,Iter,T0..3)
// the result is:

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

I will not go into details about other functions -- it is easy enough to read the descriptions/sources and play with them.
But I have one more final <del>damn cool</del> feature to tell about ;-)


## Level 80: God Mode

This feature came into my mind after reminiscing «Mortal Combat» and/or «Doom».
The way to use it came from these games: one has to press the __exact__ key sequence
like cheat codes in a game ;-) The `expand` plugin tracks single key press events
and if it finds a <del>cheat</del> magic code among the last consequently pressed keys,
it will replace it with a result.

<div class="alert alert-info" markdown="1">
#### Important Usage Notes
* every magic sequence starts and ends with a semicolon
* if you type something wrong, **you can't "fix" it** with cursor movement keys
  + delete/backspace keys!
* **you have to type «cheat codes» without "errors" from start to end**
* so remove the wrong text and start over again… repeat until you can type them 
  as if it were an unconditioned reflex ;-)
</div>

To add your own _dynamic expand function_ one has to use the `@expand.dynamic` decorator,
which accepts a compiled regular expression. If the latter matches the typed key sequence, a magic function 
will be executed and passed two parameters: a source "magic" sequence (string) and a result of `re.search()` 
(which is some 
[match object](http://docs.python.org/3/library/re.html?highlight=re.search#match-objects)). 
Here is an example of that kind of function to expand an `enum` definition:

{% highlight python %}
@expand.dynamic(re.compile('^e(?P<is_class>c)?(?P<has_type>:)?'))
@expand.jinja('enum.cpp.tpl')
@expand.postprocess
def __dynamic_enum(params, match):
    data = {}
    data['is_class'] = match.group('is_class') is not None
    data['has_type'] = match.group('has_type') is not None
    return data
{% endhighlight %}

Yeah, it is quite similar to the "usual" (jijna templated) functions except for a one more decorator ;-)

Below is a list of available dynamic snippets for C++. Every item starts with a simplified graphic image
of a regex (for visualization and better remembering ;-) that is used to match that snippet.


### `;cl;` Synopsis

![;cl; Synopsis](/assets/images/kate.cpp/cl.svg){:.img-rounded.img-responsive}

Insert a `class` declaration. Depending on the options it is capable to add a default, parameterized,
copy, move constructors and/or destructor. Also the class can be templated with a desired number of parameters.

Examples:
* `;clcd;` class with default constructor and destructor
* `;clc2;` class with constructor with 2 parameters
* `;clc1@cc@mc;` class with "conversion" constructor (single parameter), defaulted copy and move constructor/assign
* `;cl@mv-cc;` class with defaulted move constructor/assign and delete copy constructor/assign
* `;clt2vd;` class with two template parameters and a virtual destructor



### `;e;` Synopsis

![;e; Synopsis](/assets/images/kate.cpp/e.svg){:.img-rounded.img-responsive}

Examples:
* `;e;` insert C++03 enum declaration
* `;ec;` insert C++11 strong typed enum declaration
* `;ec:;` insert C++11 strong typed enum declaration with the type specified



### `;fn;` Synopsis

![;fn; Synopsis](/assets/images/kate.cpp/fn.svg){:.img-rounded.img-responsive}

Insert a function declaration or definition, if a dynamic snippet ends with `{` character.
Function may have runtime and/or template parameters, as well as various modifiers.

Examples:
* `;fnt1;` `void` function with one template parameter
* `;fn2s;` function with a `static` modifier and 2 parameters
* `;fn1vfoc;` `virtual` function with 1 parameter and `final`, `override` and `const` modifiers



### `;for;` Synopsis

![;for; Synopsis](/assets/images/kate.cpp/for.svg){:.img-rounded.img-responsive}

Generate a `for` statement with various flavors. It is capable to expand into a range-based `for`
or more "traditional" iterator-based. The latter can use either C++03 or C++11 semantics -- i.e.
when `begin()` is a member or a free function, possibly matched by ADL. The type for the range-based `for`
can be `const` and/or `ref`/`ref-ref` qualified.

Examples:
* `;fa;`  range-based `for` with the `auto` type and no body
* `;fa&&{rng;`  range-based `for` with `auto&&` type over some `rng` and an empty body
* `;fori3c{var;` loop using `const_iterators` and C++03 syntax over `var` container

### `;ns;` Synopsis

![;ns; Synopsis](/assets/images/kate.cpp/ns.svg){:.img-rounded.img-responsive}

Generate an anonymous or named (possibly nested) namespace.

Examples:
* `;ns;` insert anonymous namespace
* `;nskate::details;` insert a `kate` namespace and one nested `details`



### `;st;` Synopsis

![;st; Synopsis](/assets/images/kate.cpp/st.svg){:.img-rounded.img-responsive}

Generate a simple `struct` possibly with a given number of template parameters.



### `;sw;` Synopsis

![;sw; Synopsis](/assets/images/kate.cpp/sw.svg){:.img-rounded.img-responsive}

Generate a `switch` statement with a desired number of `case` statements and possibly with
a `default` one.



### `;try;` Synopsis

![;try; Synopsis](/assets/images/kate.cpp/try.svg){:.img-rounded.img-responsive}

Insert a `try` block with a desired number of `catch` blocks and possible a `catch (...)`.


### Dynamic Snippets Short Demo

![Dynamic Snippets Short Demo](/assets/images/kate.cpp/iddqd.gif){:.img-rounded.img-responsive}
