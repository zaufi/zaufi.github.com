---
layout: post
title: "Kate: C++/boost style (not quite) indenter"
description: "Few words about my C++ style^W indenter"
category: programming
tags: [kde, kate, C++, howto]
---
{% include JB/setup %}


Initial Motivation
==================

This indenter (initially) was designed to help **code typing** in a `boost::mpl` style
(i.e. w/ leading comma in formatted (template) parameters list). One may read rationale of such
approach in the _"C++ Template Metaprogramming: Concepts, Tools, and Techniques from Boost and Beyond"_
by David Abrahams and Aleksey Gurtovoy. It is really easy to miss a comma when invoke metafunctions and
it would leads to really complicated compile errors (a lot of). This technique help to visually control syntax
and prevent stupid errors like missed comma.

Example:

{% highlight cpp %}
typedef typename boost::mpl::fold<
    typename boost::mpl::transform_view<
        Seq
      , boost::remove_pointer<boost::mpl::_>
      >::type
  , boost::mpl::long_<0>
  , boost::mpl::eval_if<
        boost::mpl::less<boost::mpl::_2, boost::mpl::_1>
      , boost::mpl::_2
      , boost::mpl::_1
      >
  >::type type;
{% endhighlight %}

In practice I've noticed that this style can be used to format long function calls or even
`for` statements. Actually everything that can be split into several lines could be formatted that way.
And yes, it is convenient to have a delimiter (comma, semicolon, whatever) as a leading character to
make it visually noticeable.

{% highlight cpp %}

// Inheritance list formatting
struct sample
  : public base_1
  , public base_2
  , ...
  , public base_N
{
    // Parameter list formatting
    void foo(
        const std::string& param_1                      ///< A string parameter
      , double param_2                                  ///< A double parameter
      , ...
      , const some_type& param_N                        ///< An user defined type parameter
      )
    {
        // Split `for` parts into few shortter lines
        for (
            auto first = std::begin(param_1)
          , last = std::end(param_1)
          ; it != last && !is_found && !is_exit_requested
          ; ++it
          )
        {
            auto val = check_some_condition()
              ? get_then_value()
              : get_else_value()
              ;
        }
    }
};
{% endhighlight %}

It looks unusual for awhile :) but later it become (quite) "normally" and easily to read and edit :)
Really! When you want to add one more parameter to a function declaration it takes less 
typing if compare to a "traditional" way :) (especially if you have some help from an editor,
like move/duplicate the current line or a selected block up/down by a hot-key or having an indenter like this :)

Next improvements was designed to simplify typing C++ code, using most common syntactic rules, like 
_"add spaces around operators and after a comma"_, _"add space after control keywords, but not after a function name 
in a call expression"_, & etc.

Further improvements bring to the "indenter" some cute features and nowadays I'd prefer to consider it
similar to a T9 (input method) for C++ coding, but not as a _real indenter_ in contrast to others, shipped with 
[kate](http://kate-editor.org) out of the box :)
Particularly this indenter <del>exploit</del> can see "impossible" syntax and (try to) transform it to
something "reasonable" -- i.e. just like T9 it tries to be a predictive.

For example, if you have autobrackets plugin turned ON, when typing `some(nested(` gives you `some(nested(|))` w/ `|`
indicating a current cursor position. If you press `,` at this position, it gives you the following snippet
`some(nested(), |)` supposing that you want to add one more parameter to a `some()` call. While typing `;`
looks like you've done with that nested calls and gives you the `some(nested());|` snippet. Both cases help you
to avoid couple of redundant key presses ;)

... but do not even try to use this indenter to (re)format blocks of code! :)
It can do some really simple/primitive formatting, but far from good -- as I've mentioned above: this "not quite
indenter"™  designed to help you to __"do little more with less typing"__ for C++ ;)


Some features in brief
----------------------

* to start a C-style comment block type `/*` and press _ENTER_ key -- it gives you
    {% highlight cpp %}
/*
 * |
 */
{% endhighlight %}
* to start `doxygen` comment block use `/**` + _ENTER_. Every following _ENTER_ just extend the current block.
* to start C++ style comment just type `//` it will add a one space after ;)
* I prefer to have on-line-comments (i.e. C++-style comments after some expression) to be aligned at
 60th position. So typing `//` will automatically move comment to that position if there was some code before. 
 Moreover the indenter tries to keep that comments when you use _ENTER_ key to break some complex expressions 
 into a few lines   
    {% highlight cpp %}
// Before pressing ENTER at position marked by '|'
some_call_with_long parameters_list(|param1, ..., paramN);  // Comment starts here...
// After pressing ENTER: keep comment at the original line
some_call_with_long parameters_list(                        // Comment starts here...
    |param1, ..., paramN);
{% endhighlight %}
* ...also try to use _ENTER_ in the middle of a comment text ;-)
* typing `///` gives you `/// ` (with a space) or `///< ` depending on presence of code at the current line
* from time to time I use grouping in a doxygen documentation, so typing `//@` gives you:
    {% highlight cpp %}
//@{
|
//@}
{% endhighlight %}
* always add a space after `,` character -- simple, but really convenient! really soon you've stopped typing 
  a space after any comma and feel yourself inconvenient with other indenters ;)
* typing `<` without a space after some identifier adds a closing angle bracket (like notorious autobrackets 
  plugin do for other bracket types), because template instantiation guessed. So typing `std::vector<|` 
  gives you `std::vector<|>`. But `std::cout<|>` turns into `std::cout << |` after pressing one more `<` ;-)
* a lot punctuation characters being typed withing parenthesis is a some operator (or at least part of), so will be
  moved out of parenthesis (most likely call expression). Try to type various punctuators at marked position
  in the following snippet `some(|)`...
* typing a backslash in on a line of a long `MACRO` definition cause realign all others to fit to a longest line:
    {% highlight cpp %}
#define PP_FORM_A_ROW(State, Data, Elem)    \
  {                                         \
    {                                       \
        field_path(                         \
            BOOST_PP_TUPLE_ELEM(2, 0, Data) \
          , BOOST_PP_TUPLE_ELEM(2, 1, Data) \
          , BOOST_PP_TUPLE_ELEM(2, 0, Elem) \
          )                                 \
      , BOOST_PP_TUPLE_ELEM(2, 1, Elem)     \
    }                                       \
  , manage_config_keys::action::GET         \
  },
{% endhighlight %}
* typing `R"` will initiate a raw string literal

This _not-quite-indenter_™ has some other (smaller) features to reduce typing. Hope, you'll like them if found! ;)

PS: The other things I've found useful when typing C++ code can be plugged w/
[some Python plugins dedicated to C++]({% post_url 2014-02-13-kate-c++-stuff %}).
