+++
date = '2012-09-02'
title = 'Building an Eclipse Plugin with Tycho'
+++

I spent a couple of days this week converting the build of an Eclipse
plugin project from Ant with Ivy to Maven using the Tycho plugins.
I'll demonstrate the build with an example project.

First off I already have the following pre-requisites:

- Eclipse 4.2
- Eclipse [PDE](http://www.eclipse.org/pde/)
- Maven 3

### Create a plugin project in Eclipse

I created the following:

- a plugin, with some simple functionality
- a feature project (features are just [groupings of related plugins](https://www.google.co.uk/search?q=what+is+an+eclipse+feature&ie=utf-8&oe=utf-8&aq=t&rls=org.mozilla:en-GB:official&client=firefox-a&channel=fflb#hl=en&client=firefox-a&hs=UnU&rls=org.mozilla:en-GB:official&channel=fflb&q=eclipse+feature&tbs=dfn:1&tbo=u&sa=X&ei=tUpDUIT6Coqp0QWR4oCQAw&ved=0CCAQkQ4&bav=on.2,or.r_gc.r_pw.r_qf.&fp=b566ea27a82450a4&biw=1440&bih=787))
- an update site, which acts as a distributable package

Each project is a necessary part of the build: the site contains a feature,
and the feature contains one or more plugins. That's just how Eclipse
likes it.

![](http://upload.wikimedia.org/wikipedia/commons/thumb/0/0a/Russian_Dolls.jpg/800px-Russian_Dolls.jpg)

Without any further ado...

### Create a plugin

`File -> New -> Plug-in project`

I created a new plug-in project called com.example.plugin, and selected
the 'Hello, World Command' template. I've got two points to make about this:

1. Does anyone really say 'Hello, World' (with the comma)? Whenever
I read this it sounds like the beginning of a
[halting Shatnarian monologue](http://www.youtube.com/watch?v=HiFEzc_gsuw)

2. The version qualifier. DO NOT ALTER it. DO NOT! DO... NOT!

The version the plug-in wizard will assign to a new plugin is 
'1.0.0.qualifier'. It's important to know that '.qualifier' is a
token that is replaced during the build with a timestamp (or a version
scheme of your choosing, if you're patient and like a challenge).

You can fire up the project from Eclipse; it works off the bat,
and it isn't so exciting that you'll forget to carry on reading.

###Create a feature

`File -> New -> Feature project`

I called mine com.example.feature.

After clicking 'Next' I can select 'Initialize from the  plug-ins list'
and check the entry 'com.example.plugin (1.0.0.qualifier)' to include
it in my feature.

###Create a category definition

Finally, so I can install my plugin, I created a category. I created
an empty general project called com.example.site, and created a new 
category definition within it.

`File -> New -> Project`

`File -> New -> Other -> Plug-in Development -> Category Definition`

Finally, I go to the `category.xml` and open up Eclipse's spiffy
Category Manifest Editor, and add a 'New Category', which I called
com.example.category (because I was bored of guessing what I ought to
call Eclipse components), and under it added com.example.feature.

You can get the example project [here](downloads/code/TychoExampleFeature.tar.gz).

### The time has come to Mavenise

*[Don't hold back](http://www.youtube.com/watch?v=Xu3FTEmN-eg)*.

I haven't used Maven 3 before, but it was cheerfully simple
to write the build.

I found that if you try to use a version range to specify the Tycho
dependencies, rather than a property (I used `tycho-version`), the
build will explode into a bazillion lines of backtrace.

This is a regular modular Maven project. The parent pom looks like
this:

Each module's pom looks roughly the same. The packaging and artifactId
elements must be changed. I also altered the name. The plugin `pom.xml`:

The feature `pom.xml`:

And the update site `pom.xml`:

Now, running `mvn clean package` results in the warm Maven feeling:

    [INFO] ------------------------------------------------------------------------
    [INFO] Reactor Summary:
    [INFO] 
    [INFO] parentpom ......................................... SUCCESS [0.060s]
    [INFO] TychoExamplePlugin ................................ SUCCESS [1.042s]
    [INFO] TychoExampleFeature ............................... SUCCESS [0.381s]
    [INFO] TychoExampleRepository ............................ SUCCESS [2.072s]
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 15.726s
    [INFO] Finished at: Sun Sep 02 12:57:41 BST 2012
    [INFO] Final Memory: 35M/81M
    [INFO] ------------------------------------------------------------------------

*Verily, woot*.

###More to do

I've read a bunch of documentation and failed so far to get simple things
like versioning, deployment to a Maven repository, and
unit tests working. I'll follow up if these things ever materialise, 
meanwhile, here are some links:

- [Testing with Tycho](http://wiki.eclipse.org/Tycho/Packaging_Types#eclipse-test-plugin)
- [Versioning the build](http://www.eclipse.org/tycho/sitedocs/tycho-packaging-plugin/build-qualifier-mojo.html),
and a [discussion which touches on the nuances of versioning a build](http://software.2206966.n2.nabble.com/Tycho-and-feature-versioning-td3651356.html).
