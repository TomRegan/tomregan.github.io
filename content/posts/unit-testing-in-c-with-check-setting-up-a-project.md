---
title: Unit Testing in C with Check - Setting Up a Project
date: 2012-08-27
---

In a [previous post](blog/2012/08/12/unit-testing-in-c-with-check)
I went through the steps to run the
[Check](http://check.sourceforge.net/)
tutorial, had a bit of a whinge about how difficult
is had all been, and mumbled a bit about coming back when I'd figured
out how to make everything work on a real project. In fact, that
was much simpler than I expected.

At the end of this, I'll have a simple C program with a corresponding
build that uses the Check testing framework.

### Install Check

One of the nice things about Check is that you can find it in a lot
of repositories. Ubuntu, CentOS and MacPorts all have a `check`
package, but in the worst case you may have to compile it for your
platform.

I'm starting out with a fresh Ubuntu install, so I could install Check
with `apt-get`, but for the sake of it, I'm going to compile it instead.
First I needed to get a bit of a tool chain set up:

    apt-get install git build-essential curl

Then, I downloaded Check from [sourceforge.net](http://sourceforge.net/projects/check/):

    curl -L "http://downloads.sourceforge.net/project/check/check/0.9.8/check-0.9.8.tar.gz?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fcheck%2Ffiles%2F&ts=1346064997&use_mirror=switch" > check-0.9.8.tar.gz

I unzipped `check-0.9.8.tar.gz` and built it. The Check build uses
autotools, which is a build technology for people who want to
collaborate, but with people they don't like very much.
Check's other dependencies are listed in the readme.

    apt-get install automake autoconf libtool pkg-config texinfo

Then I built Check.

    autoreconf --install
    ./configure
    make
    make install

I'm mainly interested in the headers and libraries, which the installation
puts in `/usr/local/include` and `/usr/local/lib`.

During the build, I need to make sure that `/usr/local/include` is on the
include path, and `/usr/local/lib` on the library path.
If I was building manually with `make` I would use
`make -L /usr/local/lib -I /usr/local/include`. Instead I'll add this
to the makefile.

###Create a Project

I created a `project` folder containing `src`, `test`, `include` and
`lib` directories with:

    mkdir -p project/{src,test,include,lib}

and created the following root makefile in the `project` directory:

{% highlight make %}
all:
    +$(MAKE) -C src
    +$(MAKE) -C test

check:
    +$(MAKE) -C test check

clean:
    +$(MAKE) -C src clean
    +$(MAKE) -C test clean
{% endhighlight %}

I wrote a simple C function to test some implementation:

{% highlight c %}
const char*
greet(const char *name)
{
    return "";
}
{% endhighlight %}

and a corresponding header:

{% highlight c %}
const char* greet(const char *);
{% endhighlight %}

plus a main method.

{% highlight c %}
#include <stdio.h>
#include "greet.h"

int
main(void)
{
    puts(greet("Tom"));
    return 0;
}
{% endhighlight %}

The following `Makefile` lives in `src`, and is called by the root
makefile:

{% highlight make %}
program_NAME := main
program_SRCS := $(wildcard *.c)
program_OBJS := ${program_SRCS:.c=.o}

CFLAGS += -I ../include

.PHONY: all clean

all: $(program_NAME)

$(program_NAME): $(program_OBJS)
    $(CC) $(CFLAGS) -o $(program_NAME) $(program_OBJS)
    cp greet.o ../lib

clean:
    $(RM) $(program_NAME) $(program_OBJS)
{% endhighlight %}

I wrote a simple failing test with no implementation:

{% highlight c %}
#include <check.h>
#include <greet.h>

START_TEST (check_correct_greeting_is_returned)
{
    fail("test not implemented");
}
END_TEST
{% endhighlight %}

and then, a more complicated test runner. There's no requirement to
separate the test runner out from the test suites, but I prefer it.

{% highlight c %}
#include <check.h>
#include "check_main.c"

Suite *
request_suite(void)
{
    Suite *s = suite_create("request");

    TCase *tc_core = tcase_create("Core");
    tcase_add_test(tc_core, check_correct_greeting_is_returned);
    suite_add_tcase(s, tc_core);

    return s;
}

int
main(void)
{
    int number_failed;
    Suite *s = request_suite();
    SRunner *sr = srunner_create(s);
    srunner_run_all(sr, CK_NORMAL);
    number_failed = srunner_ntests_failed(sr);
    srunner_free(sr);

    return (number_failed == 0) ? 0 : 1;
}
{% endhighlight %}

I based `test_runner.c` on some example code I found in the Check
documentation which you can see [here](http://check.sourceforge.net/doc/check_html/check_3.html#SEC8).

The test `Makefile` looks like this:

{% highlight make %}
check_NAME := test_runner
check_SRCS := $(wildcard *.c)
check_OBJS := ${check_SRCS:.c=.o}
check_LIBS := -lcheck

program_OBJS := $(wildcard ../lib/*.o)

CFLAGS += -L/usr/local/lib -I/usr/local/include -I../include

.PHONY: all clean check

all: $(check_NAME)

$(check_NAME): $(check_OBJS)
    $(CC) $(CFLAGS) -o $(check_NAME) $(check_OBJS) $(program_OBJS) $(check_LIBS)

check: all
    ./test_runner

clean:
    $(RM) $(check_NAME) $(check_OBJS)
{% endhighlight %}

Now I can run `make check` to see the unit test working, i.e. failing.

    vagrant@precise64:~/Development/project$ make check
    make -C test check
    make[1]: Entering directory `/home/vagrant/Development/project/test'
    cc -L/usr/local/lib -I/usr/local/include -I../include   -c -o check_main.o check_main.c
    cc -L/usr/local/lib -I/usr/local/include -I../include   -c -o test_runner.o test_runner.c
    cc -L/usr/local/lib -I/usr/local/include -I../include -o test_runner check_main.o test_runner.o ../lib/greet.o -lcheck
    ./test_runner
    Running suite(s): request
    0%: Checks: 1, Failures: 1, Errors: 0
    check_main.c:7:F:Core:check_correct_greeting_is_returned:0: test not implemented
    make[1]: *** [check] Error 1
    make[1]: Leaving directory `/home/vagrant/Development/project/test'
    make: *** [check] Error 2

###Code some code

Finally, I did some of that programming stuff to make the program do what
I expect, which is to greet me with *["ohai Tom! xxx"](http://www.highhatstudios.com/imgs_ext/lolrats/macros/ohai.jpg)*.

{% highlight diff %}
#include <check.h>
+#include <string.h>
#include <greet.h>

START_TEST (check_correct_greeting_is_returned)
{
-       fail("test not implemented");
+       fail_unless(0 == strncmp("ohai Tom! xxx", greet("Tom"), (size_t) 255));
}
END_TEST
{% endhighlight %}

Now my test fails with an assertion error:

    vagrant@precise64:~/Development/project$ make check
    make -C test check
    make[1]: Entering directory `/home/vagrant/Development/project/test'
    ./test_runner
    Running suite(s): request
    0%: Checks: 1, Failures: 1, Errors: 0
    check_main.c:8:F:Core:check_correct_greeting_is_returned:0: Assertion '0 == strncmp("ohai Tom! xxx", greet("Tom"), (size_t) 255)' failed
    make[1]: *** [check] Error 1
    make[1]: Leaving directory `/home/vagrant/Development/project/test'
    make: *** [check] Error 2

Because [TDD](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd)
is all about writing the simplest code to pass the test, my
implementation is as follows:

{% highlight diff %}
--- a/src/greet.c
+++ b/src/greet.c
@@ -1,5 +1,5 @@
const char*
greet(const char *name)
{
-       return "";
+       return "ohai Tom! xxx";
}
{% endhighlight %}

*Merci beaucoup, et bonne nuit*.

You can download the simple project [here](downloads/code/simple-check-project.tar.gz),
or check out my pet project, [pushy, on github](https://github.com/TomRegan/pushy)
for a more realistic example.
