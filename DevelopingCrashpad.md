# Status #

Crashpad currently consists of a crash-reporting client and some related tools for Mac OS X. The core client work for Mac OS X is substantially complete. Crashpad has become the crash reporter client for [Chromium](https://dev.chromium.org/Home) on Mac OS X as of [March 2015](https://chromium.googlesource.com/chromium/src/+/d413b2dcb54d523811d386f1ff4084f677a6d089).

Crashpad is actively being bootstrapped on Windows. With the exception of the snapshot library, most major components are now working on Windows, and work is progressing on the Windows snapshot implementation.

There are plans to bring the Crashpad client to other platforms, and to add a crash report processing component to Crashpad in the future.

Crashpad is a Chromium project. In order to function on its own in other projects, it uses [mini\_chromium](https://chromium.googlesource.com/chromium/mini_chromium/), a small, self-contained library that provides many of Chromium’s useful low-level base routines. [mini\_chromium’s README](https://chromium.googlesource.com/chromium/mini_chromium/+/master/README) provides more detail.

# Prerequisites #

To develop Crashpad, the following tools are necessary, and must be present in the `$PATH` environment variable:
  * Chromium’s [depot\_tools](http://dev.chromium.org/developers/how-tos/depottools).
  * [Git](http://git-scm.com/). This is provided by Xcode on Mac OS X and by depot\_tools on Windows.
  * [Python](https://www.python.org/). This is provided by the operating system on Mac OS X, and by depot\_tools on Windows.
  * Appropriate development tools. For Mac OS X, this is [Xcode](https://developer.apple.com/xcode/), and for Windows, it’s [Visual Studio](http://www.visualstudio.com/).

# Getting the Source Code #

The main source code repository is a Git repository hosted at https://chromium.googlesource.com/crashpad/crashpad. Although it is possible to check out this repository directly with `git clone`, Crashpad’s dependencies are managed by [gclient](http://dev.chromium.org/developers/how-tos/depottools#TOC-gclient) instead of Git submodules, so to work on Crashpad, it is best to use `gclient` to get the source code.

`gclient` is part of the [depot\_tools](http://dev.chromium.org/developers/how-tos/depottools). There’s no need to install it separately.

## Initial Checkout ##

<pre>
$ mkdir ~/crashpad<br>
$ cd ~/crashpad<br>
$ gclient config --unmanaged https://chromium.googlesource.com/crashpad/crashpad<br>
$ gclient sync<br>
</pre>

## Subsequent Checkouts ##

<pre>
$ cd ~/crashpad/crashpad<br>
$ git pull -r<br>
$ gclient sync<br>
</pre>

# Building #

Crashpad uses [gyp](https://gyp.googlecode.com/) to generate [Ninja](https://martine.github.io/ninja/) build files. The build is described by `.gyp` files throughout the Crashpad source code tree. The `build/gyp_crashpad` script runs `gyp` properly for Crashpad, and is also called when you run `gclient sync` or `gclient runhooks`.

The Ninja build files and build output are in the `out` directory. Both debug- and release-mode configurations are available. The examples below show the debug configuration. To build and test the release configuration, substitute `Release` for `Debug`.

<pre>
$ cd ~/crashpad/crashpad<br>
$ ninja -C out/Debug<br>
</pre>

Ninja is part of the [depot\_tools](http://dev.chromium.org/developers/how-tos/depottools). There’s no need to install it separately.

# Testing #

Crashpad uses [Google Test](https://googletest.googlecode.com/) as its unit-testing framework, and some tests use [Google Mock](https://googlemock.googlecode.com/) as well. Its tests are currently split up into several test executables, each dedicated to testing a different component. This may change in the future. After a successful build, the test executables will be found at `out/Debug/crashpad_*_test`.

<pre>
$ cd ~/crashpad/crashpad<br>
$ out/Debug/crashpad_minidump_test<br>
$ out/Debug/crashpad_util_test<br>
</pre>

A script is provided to run all of Crashpad’s tests. It accepts a single argument that tells it which configuration to test.

<pre>
$ cd ~/crashpad/crashpad<br>
$ python build/run_tests.py Debug<br>
</pre>

# Contributing #

Crashpad’s contribution process is very similar to [Chromium’s contribution process](http://dev.chromium.org/developers/contributing-code).

## Code Review ##

A code review must be conducted for every change to Crashpad’s source code. Code review is conducted on [Chromium’s Rietveld](https://codereview.chromium.org/) system, and all code reviews must be sent to an appropriate reviewer, with a Cc sent to [crashpad-dev](https://groups.google.com/a/chromium.org/forum/#!forum/crashpad-dev). The `codereview.settings` file specifies this environment to `git-cl`.

`git-cl` is part of the [depot\_tools](http://dev.chromium.org/developers/how-tos/depottools). There’s no need to install it separately.

<pre>
$ cd ~/crashpad/crashpad<br>
$ git checkout -b work_branch origin/master<br>
…do some work…<br>
$ git add …<br>
$ git commit<br>
$ git cl upload<br>
</pre>

Uploading a patch to Rietveld does not automatically request a review. You must select a reviewer and mail your request to them (with a Cc to crashpad-dev) from the Rietveld issue page after running `git cl upload`. If you have lost track of the issue page, `git cl issue` will remind you of its URL. Alternatively, you can request review when uploading to Rietveld by using `git cl upload --send-mail`

Git branches maintain their association with Rietveld issues, so if you need to make changes based on review feedback, you can do so on the correct Git branch, committing your changes locally with `git commit`. You can then upload a new patch set with `git cl upload` and let your reviewer know you’ve addressed the feedback.

## Landing Changes ##

After code review is complete and “LGTM” (“looks good to me”) has been received from all reviewers, project members can commit the patch themselves:

<pre>
$ cd ~/crashpad/crashpad<br>
$ git checkout work_branch<br>
$ git cl land<br>
</pre>

Crashpad does not currently have a [commit queue](http://dev.chromium.org/developers/testing/commit-queue), so contributors that are not project members will have to ask a project member to commit the patch for them. Project members can commit changes on behalf of external contributors by patching the change into a local branch and landing it:

<pre>
$ cd ~/crashpad/crashpad<br>
$ git checkout -b for_external_contributor origin/master<br>
$ git cl patch 12345678  # 12345678 is the Rietveld issue number<br>
$ git cl land -c 'External Contributor <external@contributor.org>'<br>
</pre>

## External Contributions ##

Copyright holders must complete the [Individual Contributor License Agreement](https://developers.google.com/open-source/cla/individual) or [Corporate Contributor License Agreement](https://developers.google.com/open-source/cla/corporate) as appropriate before any submission can be accepted, and must be listed in the `AUTHORS` file. Contributors may be listed in the `CONTRIBUTORS` file.

# Buildbot #

The [Crashpad Buildbot](https://build.chromium.org/p/client.crashpad/) performs automated builds and tests of Crashpad. Before checking out or updating the Crashpad source code, and after checking in a new change, it is prudent to check the Buildbot to ensure that “the tree is green.”