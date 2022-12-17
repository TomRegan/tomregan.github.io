---
title: Writing a Simple Gradle Build
date: 2013-05-05
---

'Build as code' is one of those dreadful smells you start to recognise after
a while, so gradle, which is a highly free-form system, is not my first choice
for complex builds, but it is a beautiful way to build a simple set of modules.
I've written something along the lines of the following build enough times for
me to make a note of it for future copy-pasting, so here it is.

{% highlight groovy %}
apply plugin: 'base'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4+'
}
{% endhighlight %}

And run with `gradle check` for tests or `gradle assemble` to create a jar.
Gradle will assemble the jar in `build/libs`.

Executable jars need extra configuration to create a Main-Class entry in
`MANIFEST.MF`.

{% highlight diff %}
+jar {
+    manifest { attributes 'Main-Class': 'io.github.tomregan.Application' }
+}
{% endhighlight %}

And to build a number of modules, stick `settings.gradle` in the modules` root,
which would look like this:

{% highlight groovy %}
include 'module-core'
include 'module-demo-app'
{% endhighlight %}

If you had the following.

    modules
    |- module-core
    |  `- build.gradle
    |- module-demo-app
    |  `- build.gradle
    `- settings.gradle

There's loads of not very good documentation at [gradle.org](https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&ved=0CDMQFjAA&url=http%3A%2F%2Fwww.gradle.org%2Fdocumentation&ei=6y2GUZWJN8OZhQeO4ICIBQ&usg=AFQjCNElnzzs-KfHqp0WfL-zuo90jM3Kuw&sig2=KsvVY1h0a7zlTydwiA71TA&bvm=bv.45960087,d.ZG4),
but the best source of information is the build of gradle itself, which is
available [over on github](https://github.com/gradle/gradle).
