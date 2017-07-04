---
layout: post
title: "Extend pytest with fixures"
description: "Few more helpful pytext fixrures"
category: programming
tags: [programming python]
---
{% include JB/setup %}

A few helper functions
======================

It is quite common practice to store some data required for test in a repository
as indicidual files. First of all it could be some input data for tests. Secondly,
files to check a test output somehow.

I've found the following bunch of functions are pretty useful:

~~~ python
import pathlib

def data_dir():
    return pathlib.Path(__file__).parent / 'data'

def output_dir():
    return data_dir() / 'output'

def expected_results_dir():
    return data_dir() / 'expected-results'

def make_data_filename(filename):
    return data_dir() / filename
~~~

I put 'em to `test/conftest.py` usually.


A simple way to assert test output
==================================

Since begining of my experience with [pytest](https://docs.pytest.org/en/latest/) I've looked for an easy way
to match test output against an expected text or pattern. When I found nothing in docs and nothing interesting in
StackOverflow and Google, I desided to write my own wheel ;-)


Add CLI option to store patterns
--------------------------------

I like [this featue][save-patterns] in Boost Unit Testing Framework: `--save_pattern` CLI option instead of c
hecking the captured output result, just store it into a file, which is usually kept in a VCS as _expected_ result.

Now lets implement the same for pytest! ;-) I found a [sample][pytest-addoption], so it was the easiest part:

~~~ py
# Add `--save-patterns` CLI boolean option
def pytest_addoption(parser):
    parser.addoption(
        '--save-patterns'
      , action='store_true'
      , help='store matching patterns instead of checking them'
      )
~~~

<div class="alert alert-info" markdown="1">
#### Attention

This function (and everything else in this article) must be defined in `conftest.py` file
of your tests. Then pytest can find your extensions and you could use them in your tests.
</div>

So far, so good... but by itself this option do nothing...


Write a matching fixture
------------------------

In fact it would be a [named argument fixture][fun-as-arg]. What I want is a simple as possible way to
match a test output with some expectations. Good to know that pytest can [capture][capfd] test output for you.
Then (close to the end of your test) you can read it:

~~~ py
    out, err = capfd.readouterr()
~~~

and have a text strings. But in my experience:
* test's output could be really long (to `assert out == 'expected string`)
* output may change from run to run (e.g. timestamps in logged strings)

To address the first issue it would be nice to store an expected output in a file under VCS control.
For the second case, stored expectation could be a regular expression to be matched against the test's output.

How it may looks like?

~~~ py
def my_test(capfd, expected_out):
    # ... do some testing ...

    out, err = capfd.readouterr()

    assert expected_out == out
~~~


Ok, lets see how `expected_out` would look like... First of all it is a fixture name -- i.e. a function accepting
[`request` parameter][fixture-request] provided by pytest. Using `resuest` it's possible to get a module/class/function
name which requested this fixture. Lets use 'em to form a path to expectations file, which is in turn located somewhere
in `test/data/expected-results/` directory of project sources. The next part is to make an equal comparable object
and return as a fixture "result". Equal operator later will be used in `assert` expression, where the second expression
is the captured output (string type).

Ok, here is a full code:

~~~ py
import pathlib
import pytest
import re
import warnings

class _content_check_or_store_pattern:

    def __init__(self, filename, store):
        self._filename = filename
        self._store = store


    def __eq__(self, text):
        if self._store:
            self._store_pattern_file(text)

        if not self._filename.exists():
            warnings.warn('There is no file to match `{}`'.format(self._filename), RuntimeWarning)
            return False

        expected_text = self._filename.read_text().strip()
        return expected_text == text


    def match(self, text):
        if self._store:
            self._store_pattern_file(text)
            return True

        if not self._filename.exists():
            warnings.warn('There is no file to match `{}`'.format(self._filename), RuntimeWarning)
            return False

        content = ' '.join(self._filename.read_text().strip().splitlines())
        what = re.compile(content)
        transformed_text = ' '.join(text.splitlines())
        return bool(what.match(transformed_text))


    def _store_pattern_file(self, text):
        assert self._store, 'Code review required!'
        if not self._filename.parent.exists():
            self._filename.parent.mkdir(parents=True)

        self._filename.write_text(text)


def _make_expected_filename(request, ext):
    result = expected_results_dir()

    if request.cls is not None:
        result /= request.cls.__name__

    result /= request.function.__name__ + ext

    return result


@pytest.fixture
def expected_out(request):
    return _content_check_or_store_pattern(
        _make_expected_filename(request, '.out')
      , request.config.option.save_patterns         # The value of `--save-patterns` CLI option
      )


@pytest.fixture
def expected_err(request):
    return _content_check_or_store_pattern(
        _make_expected_filename(request, '.err')
      , request.config.option.save_patterns
      )
~~~

Hence in a test function `my_test`, `expected_out` is an instance of `_content_check_or_store_pattern`,
so one can use `assert expected_out == out`! For the fist time it is recommended to run `py.test` with
`--save-patters` option and particular test to execute -- just to overwrite only one expectations file
(and do not touch others):

~~~
$ pytest --save-patterns test/some_test.py::my_test
~~~

If test output has changed parts, then a better way is to use `match()` method:

~~~ py
def my_test(capfd, expected_out):
    # ... do some testing ...

    out, err = capfd.readouterr()

    assert expected_out.match(out)
~~~

Run `pytest` to store initial pattern, then edit the `test/data/expected-results/my_test.out` to replace
mutable parts with `.*` regex's wildcards (or some other, more precise expressions).


[save-patterns]: http://www.boost.org/doc/libs/1_64_0/libs/test/doc/html/boost_test/utf_reference/rt_param_reference/save_pattern.html
[pytest-addoption]: https://docs.pytest.org/en/latest/example/simple.html
[fun-as-arg]: https://docs.pytest.org/en/latest/fixture.html#fixtures-as-function-arguments
[capfd]: https://docs.pytest.org/en/latest/capture.html
[fixture-request]: https://docs.pytest.org/en/latest/builtin.html#_pytest.fixtures.FixtureRequest

