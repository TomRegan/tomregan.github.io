+++
date = '2012-09-07'
title = 'Monitor a Long Running Process with Control T'
+++

One of the nice things about being in Linux exile is learning about
new things that I hadn't noticed were missing from my life, like
being able to tell if a process has hung, or it's just taking a
really long time.

    tom@binky:~:$ cp Downloads/FreeBSD-9.0-RELEASE-i386-disc1.iso .
      ...
      ^t
      ...
    load: 0.17  cmd: cp 20253 running 0.00u 0.16s
    Downloads/FreeBSD-9.0-RELEASE-i386-disc1.iso -> ./FreeBSD-9.0-RELEASE-i386-disc1.iso  35%

Hitting Control-t while waiting for a process to complete outputs,
and 'scuse me while I quote from the manpage:

> the current load average, the name of the command in the foreground,
> its process ID, the symbolic wait channel, the number of user and
> system seconds used, the percentage of cpu the process is getting,
> and the resident set size of the process.

It's basically all there, although I don't see the resident size on
Mac OSX (where I took the manpage snippet from).
