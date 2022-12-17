---
title: Unit Testing in C with Check
date: 2012-12-09
---

###Pushy
I've been developing a cheerfully indolent little HTTP server for some
time now, as one of those 'learn-more-c', 'learn-TDD' projects that
I imagine nearly everyone has lying around, half neglected.

You can see the code for [pushy on github](https://github.com/TomRegan/pushy).

I've also been familiarizing myself with the HTTP RFCs as I go, so I was
planning to add a <a id="reverse-pooky"></a>[414 response](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.15)^[[1](#pookie)]
to pushy's repertoire, which I reckoned
would take me about 30 minutes, but I have the TDD policy on this
project, so _naturally_ I can't write a line of code until there's
a failing test. I'm on a new machine, so I need to install 
[Check](http://check.sourceforge.net/).

Let the games commence.

###Install MacPorts

MacPorts is available from the [MacPorts project website](http://www.macports.org/).
Once you have this, installing Check should be as simple as `port install check`.

###Get the tutorial project

You'll be pleased to know that Check comes with a _wildly_ overcomplicated 
tutorial.

On my system the Check install includes a set of tutorial files in 
`/opt/local/share/doc/check/example`. If you don't find anything here
it should be possible to query your package manager, for instance
`port contents check`, or ` rpm -ql <package-name>`,
`dpkg -L <package-name>`, depending on your poison.

Create a working directory and copy the tutorial files there:

    mkdir check-tutorial
    cp -r /opt/local/share/doc/check/example check-tutorial/

###Build the project

That's easy, isn't it? I checked the README, and saw I needed to
grab the following dependencies:
* Autoconf
* Automake
* Libtool

And then it should be a case of:

    autoreconf --install
    ./configure
    make
    make check

So, I gave that a spin:

    tom@binky:check-tutorial:$ autoreconf --install
    glibtoolize: Consider adding `AC_CONFIG_MACRO_DIR([m4])' to configure.ac and
    glibtoolize: rerunning glibtoolize, to keep the correct libtool macros in-tree.
    glibtoolize: Consider adding `-I m4' to ACLOCAL_AMFLAGS in Makefile.am.
    automake: warnings are treated as errors
    /opt/local/share/automake-1.12/am/ltlibrary.am: warning: 'libmoney.la': linking libtool libraries using a non-POSIX
    /opt/local/share/automake-1.12/am/ltlibrary.am: archiver requires 'AM_PROG_AR' in 'configure.ac'
    src/Makefile.am:3:   while processing Libtool library 'libmoney.la'
    tests/Makefile.am:5: warning: compiling 'check_money.c' with per-target flags requires 'AM_PROG_CC_C_O' in 'configure.ac'
    autoreconf: automake failed with exit status: 1

And fell very much at the first hurdle. Still, those are some sensible
errors so it should be easy to fix. First I added the missing macros, as
directed by the output, which led to the following changes to `Makefile.am`:

{% highlight diff %}
--- /opt/local/share/doc/check/example/Makefile.am 2012-08-10 22:56:02.000000000 +0100
+++ Makefile.am 2012-08-12 15:21:39.000000000 +0100
@@ -1,5 +1,3 @@
 ## Process this file with automake to produce Makefile.in
  
+SUBDIRS = src . tests
+
+ACLOCAL_AMFLAGS = -I m4
-SUBDIRS = src . tests
{% endhighlight %}

I also added `AC_CONFIG_MACRO_DIR([m4])` to `configure.ac` and created
the corresponding directory in the project:

    tom@binky:check-tutorial:$ mkdir m4

Now I get:

    tom@binky:check-tutorial:$ autoreconf --install
    ...
    /opt/local/share/automake-1.12/am/ltlibrary.am: archiver requires 'AM_PROG_AR' in 'configure.ac'

So I add the final macro to `configure.ac` which results in the
following changes:

{% highlight diff %}
--- /opt/local/share/doc/check/example/configure.ac 2012-08-10 22:56:02.000000000 +0100
+++ configure.ac 2012-08-12 15:24:38.000000000 +0100
@@ -10,16 +10,13 @@
# place to put some extra build scripts installed
AC_CONFIG_AUX_DIR([build-aux])

+AC_CONFIG_MACRO_DIR([m4])
+
# fairly severe build strictness
# change foreign to gnu or gnits to comply with gnu standards
AM_INIT_AUTOMAKE([-Wall -Werror foreign 1.9.6])

# Checks for programs.
+AM_PROG_CC_C_O
-AC_PROG_CC
AC_PROG_LIBTOOL
+AM_PROG_AR

# Checks for libraries.
{% endhighlight %}

And finally it passes this step.

The next instruction is `./configure`, which in my experience is _always_ 
a one-stop-shop for merriment. It wasn't too bad in this instance.

    checking for check - version >= 0.8.2... no
    *** Could not run check test program, checking why...
    *** The test program failed to compile or link. See the file config.log for
    *** the exact error that occured.
    configure: error: check not found

This is to be expected, because my environment doesn't know about the
includes MacPorts installs under `/opt`. To fix, I first check the variable
`C_INCLUDE_PATH` is unset:

    echo $C_INCLUDE_PATH

And then set it:

    export C_INCLUDE_PATH=/opt/local/include

If it already has some value, you should prepend the new path to the old,
i.e.

    export C_INCLUDE_PATH=/opt/local/include:$C_INCLUDE_PATH

And then, run `./configure` a final time. Next, `make`. If I were a betting
man I'd put money on the linker having a great big episode.

    tom@binky:check-tutorial:$ make; make check
    ...
    /bin/sh ../libtool --tag=CC   --mode=link gcc  -g -O2   -o check_money check_money-check_money.o ../src/libmoney.la -lcheck 
    libtool: link: gcc -g -O2 -o .libs/check_money check_money-check_money.o  ../src/.libs/libmoney.dylib -lcheck
    ld: library not found for -lcheck
    collect2: ld returned 1 exit status
    make[2]: *** [check_money] Error 1
    make[1]: *** [check-am] Error 2
    make: *** [check-recursive] Error 1

I love it when I'm right. No, really. Love it.

So the most useful line in there is `ld: library not found for -lcheck`,
which tells me the linker (`ld`) can't find `libcheck`. Because I'm using
a Mac I know that I'm looking for the `.dylib` files that were installed
with Check. On linux I expect they would be `.so` files. In my case
these happen to be in `/opt/local/lib`, so I need to make the environment
aware of this location by setting `LIBRARY_PATH` (linux: `LD_LIBRARY_PATH`):

    export LIBRARY_PATH=/opt/local/lib

And finally:

    tom@binky:check-tutorial:$ make check
    Making check in src
    ...
    Running suite(s): Money
    100%: Checks: 3, Failures: 0, Errors: 0
    PASS: check_money
    =============
    1 test passed
    =============

Job done.

### Why's it so painful?
The tutorial fails at the first step and it can be hard to understand why.
I've never had the pleasure of using autotools, but I'm beginning to see why there's some
anti-gnu sentiment in the Unix-sphere. It's a tremendously complicated
build for a very, very simple project.

It doesn't help that many of the links to documentation on the Check
project page are [broken](http://check.sourceforge.net/doc/check_html/index.html).

I happen to think
that a test framework should be independent of the build system
([you may say I'm a dreamer](http://www.youtube.com/watch?v=yRhq-yO1KN8)),
so I'm going to pursue this to some sort of logical, probably painful,
conclusion.

###What have we achieved?
Well, nothing that I set out to achieve, which was having a working
test suite so I can add a simple feature to my server. Expect a
_[Part Deux](http://www.youtube.com/watch?v=hEc-9hIdK0E)_
in the near-ish future.

<a id="pookie"></a>
<sup>[[1](#reverse-pooky)]</sup> _and yes, this is related to a buffer over-run bug, thank you for asking._
