---
layout: post
title: "Integrate your Boost UnitTests with TeamCity"
description: "How to report test results to TeamCity"
category: programming
tags: [teamcity, boost, unit-test]
---
{% include JB/setup %}

A long time ago I've started to use [`teamcity-cpp`][official]. It became broken with boost 1.59 release, cuz latter
had updated version of Unit Test Framework. That is a moment when I started my [fork][my-fork], 
cuz this code was actively used by a lot of configurations.

I've added a way to build this project with CMake (and this PR was accepted to upstream), but sooner I've became fail to submit further patches.
Anyway, I continued to work and now my fork has some significant differences <del>which make it impossible to contribute to upstream</del>.

The most important feature is a CMake finder module for Boost UTF and support for CMake based projects. To use it one should find the package
first (as it usually done for third party libraries/software):

    # Find Boost UTF...
    find_package(Boost REQUIRED COMPONENTS unit_test_framework)

    # ... and corresponding TeamCity integration library
    find_package(teamcity-cpp-boost REQUIRED)

Now you are ready to add an executable with unit tests:

    add_executable(
        unit-tests-binary
        $<TARGET_OBJECTS:teamcity-cpp-boost>
        # Your other sources here
        ...
      )

The integration package provides a small [object library][cmake-obj-lib] called `teamcity-cpp-boost` to be added to every unit tests binary
(using `TARGET_OBJECTS` [generator expression][ge]). And do not forget to link your binary with Boost UTF (as usual).

    target_link_libraries(
        unit-tests-binary
        ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY}
        # Your other required libraries here
        ...
      )

<div class="alert alert-info" markdown="1">
#### Under the hood...

It registers a [_global fixture_][gf], which is looking into defined environment variables check for TeamCity presense
and if so, it'll register a [_log formatter_][lf] which would use [_TeamCity service messages_][tc-srv-msg] to report
about testing progress...
</div>

Some other features of my fork:

* conditional support for testing libraries (the others are Google Test and CppUnit)
* simplified way to build TeamCity service messages and some other refactorings
* easy way to update test patterns (for developers of this package only)

<div class="alert alert-success" markdown="1">
#### For Gentoo Users

… you may get [teamcity-cpp-1.6.ebuild][ebuild] from my repository (overlay).
</div>


[official]: https://github.com/JetBrains/teamcity-cpp
[my-fork]: https://github.com/zaufi/teamcity-cpp
[cmake-obj-lib]: https://cmake.org/cmake/help/v3.4/command/add_library.html#object-libraries
[ge]: https://cmake.org/cmake/help/v3.4/manual/cmake-generator-expressions.7.html#informational-expressions
[gf]: http://www.boost.org/doc/libs/1_60_0/libs/test/doc/html/boost_test/utf_reference/test_org_reference/test_org_boost_global_fixture.html
[lf]: http://www.boost.org/doc/libs/1_60_0/libs/test/doc/html/boost/unit_test/unit_test_log_formatter.html
[tc-srv-msg]: https://confluence.jetbrains.com/display/TCD9/Build+Script+Interaction+with+TeamCity
[ebuild]: https://github.com/zaufi/zaufi-overlay/blob/master/dev-cpp/teamcity-cpp/teamcity-cpp-1.6.ebuild

*[UTF]: Unit Test Framework

