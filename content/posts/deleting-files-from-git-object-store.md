+++
date = '2013-04-17'
title = 'Deleting Files from Git Object Store'
+++

Tonight I accidentally did something stupid and committed binaries to
my git repository, despite knowing better in the first place. After
the commit I tried to push.

I hadn't even noticed my mistake until I came back to my terminal after 
10 minutes and saw

    Writing objects:  42% (16/38), 10.02 MiB | 21 KiB/s

Unfortunately simply `git rm`-ing the files is no good, because they're
in the object store at this point, which is essentially git's historical
record&mdash; its database of versions. I need to go back through the history
and remove all traces of the binaries in order to shrink the object
store back down to a reasonable size.

Git's [filter-branch](https://www.kernel.org/pub/software/scm/git/docs/git-filter-branch.html)
command is designed to do this. It's pretty complicated and not particularly
well documented, but if I do this...


    git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch <big-binary-mess/>'

it obligingly hoovers up my big binary mess for me. Then I can either add
an exclusion in `.gitignore`, or `git rm` the offending blobs.

Done.
