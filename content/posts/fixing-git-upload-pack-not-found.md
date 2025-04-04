+++
date = '2012-09-08'
title = 'Fixing Git Upload Pack Not Found'
+++

I've only just realised that blogging about problems is a great way of
delaying doing the thing I intended to get done in the first place.

I've just changed the location of git repositories on my server
so that instead of living in a directory belonging to the user
'tom', they now live in the home directory belonging to 'git'.

Basically, it means when I create a new repository on my server
I don't have to remember:

    git remote add origin tonchan:Development/git/path/to/things/zomg/long/

But after setting up the new user and copying the repositories, I tried
to alter the remote to point at the new location:

    tom@binky:custard:master$ git remote add origin git@tonchan:custard.git
    ash: git-upload-pack: not found
    fatal: The remote end hung up unexpectedly

¿Qué. Well `git` is most certainly on the path for the user 'git', but
the non-login shell clearly loads its path from somewhere else.
To test this out I tried:

    ssh git@tonchan echo \$PATH

Sure enough:

    /usr/bin:/bin:/usr/sbin:/sbin:/usr/syno/bin

Git isn't in any of those places.

There is an easy way to fix this I found in the ssh man page:

> Additionally, ssh reads ~/.ssh/environment, and adds lines of the format
> "VARNAME=value" to the environment if the file exists and users are
> allowed to change their environment.

So I created the file `~/.ssh/environment` and set the correct
path in it. Presto, git was happy immediately.

    tom@binky:custard:master$ git remote show origin
    * remote origin
      Fetch URL: git@tonchan:custard.git
      Push  URL: git@tonchan:custard.git
      HEAD branch: master
      Remote branch:
      master tracked
      Local branch configured for 'git pull':
      master merges with remote master
      Local ref configured for 'git push':
      master pushes to master (up to date)
