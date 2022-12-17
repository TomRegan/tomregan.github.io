---
title: Modular Projects with Bash
date: 2012-07-19
---

I try to use bash scripts wherever I can over ruby or python. Partly
that's down to stubbornness, but it's mostly because the problems I'm
trying to solve can be tackled with the tools already available
in UNIX.

One of my past sprawling bash projects was a tool that
I used for simple, repetitive tasks, like creating new source files
from templates, creating working tree structures for projects,
running unit tests, and so on.

By the time I'd got to university it had stopped working. When I tackled
the code it was a mess: far beyond human comprehension. The
codebase was a nest of `source`s and global functions, buried spaghetti
sed. I had to let it go, and since then I've avoided bash for projects 
I thought might get large.

But recently I missed my old project tool kit and decided to create a 
new one. I know bash is absolutely the right choice for each of the 
individual tasks. The problem is how to tie them all together.

When I thought about it I figured out I was only missing two important
features: the ability to group code into modules or packages, and the 
ability to contain functions in a namespace. So after an evening's work I 
came up with this, which provides a very simple set of functions for 
managing individual scripts as modules.

{% highlight bash %}
#! /bin/bash

if [ -z "$INSTALL_LOCATION" ];then
    msBaseDirectory="./"
else
    [ -d "$INSTALL_LOCATION" ] && msBaseDirectory=$INSTALL_LOCATION
    [ -z "${msBaseDirectory}" ] && echo "ERROR: cannot determine the location of modules"
fi

# use to source a module
require() {

    local sModule=$msBaseDirectory${1//.//}
    if [ -d $sModule ]; then
        source $sModule/_init
    elif [ -f $sModule ]; then
        source $sModule
    else
        echo "ERROR: could not load module '$sModule'"
        exit
    fi
}

# use to declare a module
module() {

    local sModule=$1

    # transform the module name to a path
    gsWorkingDir=${sModule//.//}
}

# use to declare an object
object() {

    local sObject=$1

    # create a constructor for the object
    eval "${sObject}.new() { __new ${sObject} ${gsWorkingDir}; };"
}

__new() {

    local sObject=$1
    local sWorkingDir=$2

    for it in ${msBaseDirectory}/${sWorkingDir}/*; do

        # create functions from the module's contents
        local sFunction=$(basename $it)
        eval "${sObject}.${sFunction}() { source $it; };"
    done
}
{% endhighlight %}

The idea is this: a directory becomes a bash module by 
virtue of having a file called `_init`. The init file should declare a 
module and an object, for example:

{% highlight bash %}
#! /bin/bash
module foo.bar.baz

object Baz
{% endhighlight %}

The object serves as a namespace for each of the scripts in the module,
so it's possible to call `Baz.func`, and whatever code is present
in the file `foo/bar/baz/func` will be executed as if it were a function.

I can make use of these functions by `require`-ing a module, calling
`new` on its object (which avoids sourcing files unnecessarily), and
making method calls on it, as follows:

{% highlight bash %}
#! /bin/bash
require foo.bar.baz

Baz.new
Baz.func
{% endhighlight %}

This executes the script in foo/bar/baz/func, it accepts arguments as
would a regular function, and variables scoped with `local` behave as you
would expect.

So far this has been a really neat solution for organizing a large
bash project, but I won't be surprised to find limitations in the future.

Some example source is available [here](downloads/code/example.tgz).
