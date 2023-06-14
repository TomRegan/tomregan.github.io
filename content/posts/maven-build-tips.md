---
title: Quick, Repeatable, Maintainable Maven Builds
date: 2023-04-25
cover:
  image: "https://hips.hearstapps.com/hmg-prod/images/construction-toys-1672760698.jpg"
  alt: "A child playing in sand with a digger"
---

There are few things more frustrating than running into issues with your
build tool. One moment your build runs smoothly, and the next it fails
inexplicably. Or maybe you've been typing `mvn clean package` for years
without really knowing why. In this post, I'll share some best
practices for using Maven more effectively, and attempt to demystify
some of its features.

### Understand the Clean Plugin

Running the Clean plugin during every build will spend unnecessary time
on recompilation and packaging. The Clean plugin is used to delete the
output of a previous build. Using it only when you need it could save
you a little time every day; so when do you need it? Maven recompiles
source files, but it won't clean up the class files of deleted and
renamed source files for you. If you delete or rename a source or a
resource file, it's a good idea to run a clean build to avoid
compilation errors, and to prevent old files from being bundled into
packages. If you run `mvn clean` every build, you delete everything,
including class files with unchanged source files, which means a longer
rebuild.

### Run Verify After Pulling Code or Changing Branches

Before you start working on new code, make sure you can build it.
Use`mvn verify` to ensure that everything is in working order. Verify
runs the entire build including integration tests, which is why the
documentation describes it as [the best default way to run a
build](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#usual-command-line-calls:~:text=If%20you%20are%20uncertain%20what%20you%20want%2C%20the%20preferred%20phase%20to%20call%20is).
Be sure to run clean after you pull or switch branches to get rid of any
old class files you have that may not belong in the new source tree.

### Review Build Warnings

Check and fix any build warnings that pop up, including seemingly small
things such as encoding problems or undefined versions. These issues can
easily sneak into a project, and fixing them can prevent problems from
piling up and help avoid unpleasant surprises down the road. It's
difficult to see all the warnings in a stream of output, so here's a
tip: set the log level to WARN and run the build.

``` p1
mvn -Dorg.slf4j.simpleLogger.defaultLogLevel=warn compile
```

### Don't Depend on Transitive Dependencies

Explicitly declare any dependencies that your code uses, rather than
relying on transitive dependencies. Transitive dependencies (the
dependencies of your dependencies) are usually available on your
project's classpath, making it tempting to reach-in and use them.
Don't, as this hurts build repeatability. When several versions of the
same dependency exist, it's likely you won't be aware of where your
dependency is coming from, and it's almost certain the version will
change upstream without your knowledge, possibly stoping your code from
compiling. Avoid this by explicitly declaring all required dependencies
in your project's POM. You can go further, and [ban transitive
dependencies](https://maven.apache.org/enforcer/enforcer-rules/banTransitiveDependencies.html)
with the Maven Enforcer Plugin.

### Create Custom Plugins

While Maven comes with a large set of default plugins, sometimes you
need more. Instead of scripting, take some time to learn to build custom
plugins. By creating plugins you'll have more control over your build
process, and you can make them into reusable components. Maven plugins
[can exist in a multi-modular
project](https://maven.apache.org/guides/mini/guide-multiple-modules.html),
or they can be published to a repository.

### Finally, Embrace Maven Conventions

Maven has a set of conventions and best practices that are widely used
in projects. These conventions cover everything from project structure
to naming, and they're designed to make Maven projects more
maintainable and portable. While it is sometimes tempting to deviate
from these conventions, you're better off sticking to them. When you
follow the conventions, you'll have fewer battles with the tool, and
your project will be more recognisable to other Maven users. You would
struggle to find better examples of Maven projects than [the Maven
project itself on GitHub](https://github.com/apache/maven-plugin-tools).
Using Maven effectively requires a bit more than just knowing the
commands. Other build tools emphasise customisation over everything
else, Maven focusses on maintainability, reliability, and reusability.
Nobody wants an exciting build, and keeping builds boring makes Maven
the best available tool for Java projects.
