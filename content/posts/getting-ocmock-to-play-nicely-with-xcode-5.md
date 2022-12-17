---
title: Getting OCMock to play nicely with XCode 5
date: 2013-11-17
---

I've started writing a dashboard Cocoa app, and I've found the process of
setting up OCMock really unreasonably painful.

### The Setup

I got the OCMock framework from [ocmock.org](http://ocmock.org/download/).

In my case I want the OSX directory for my Cocoa app, so I drag and drop
`OCMock.framework` from the `dmg` onto the Frameworks group in the project
navigator. This triggers a dialog to configure it.

![Copying the framework](/assets/images/2013-11-17-Getting-OCMock-to-play-nicely-with-XCode-5/copying.png)

The important things to note are:

* *Copy items into destination group's folder (if needed)* is checked
* *Create groups for any added folders* doesn't change
* *Add to targets* has the test target selected, and not the application

I'm now able to use OCMock in my tests.

![Importing OCMock](/assets/images/2013-11-17-Getting-OCMock-to-play-nicely-with-XCode-5/importing.png)

### Running the Tests

I get an unexpected surprise trying to run the tests however, in the form
of a `no suitable image found` error. The non-obvious solution to this is to
add a new build phase to the test target which will copy the OCMock library
into the runtime bundle.

![Adding a build phase](/assets/images/2013-11-17-Getting-OCMock-to-play-nicely-with-XCode-5/addbuildphase1.png)

Next select the framework to add to the new phase.

![Selecting the framework](/assets/images/2013-11-17-Getting-OCMock-to-play-nicely-with-XCode-5/addbuildphase2.png)

The target should look like this:

![New build target](/assets/images/2013-11-17-Getting-OCMock-to-play-nicely-with-XCode-5/addbuildphase3.png)

Again, the important points are:

* *Destination* is *Products Directory*
* *Subpath* is empty
* *Copy only when installing* is not checked

Finally, I moved the new phase run directly after compile. Running the tests
now works.

### What I Have Learned

* I can use the XCode build system now! (It's more horrible than I imagined.)
* The build system for XCode 5 is still broken
* There's an interesting tool called [cocoapods](http://cocoapods.org/)
which looks interesting, but I couldn't make it work
