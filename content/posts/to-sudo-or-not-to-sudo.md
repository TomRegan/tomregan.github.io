---
title: To Sudo or not to Sudo
date: 2012-08-25
---

This is a pretty neat trick I've just seen in a tutorial from the
chaps at [opscode](http://wiki.opscode.com/).

I've got a command whose output needs to be processed using `sudo`
(a realistic example is `curl`ing down a file to install); 
I've got at least three principals that apply in this situation:

1. only use `sudo` if privileges need to be escalated
2. waste not a minute
3. one line *all of the things*

So:

    sudo true && curl files.example.com/install.bin | sudo install.bin

It's probably fairly clear, but `true` returns 0 to the shell (`false`,
as you might guess returns 1), so the semantics go: *if sudo is available,
then download the file and install, or else do nothing*.

Much nicer than failing after a long download.
