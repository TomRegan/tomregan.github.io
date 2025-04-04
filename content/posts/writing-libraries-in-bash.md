+++
date = '2013-11-19'
title = 'Writing Libraries in Bash'
+++

<!--
![BY-NC-SA shells by Cody Geary](https://farm1.staticflickr.com/119/308384875_5232c39c9a_n.jpg)
-->

If you're a Python programmer, chances are you're familiar with the idiom:

```python
if __name__ == '__main__':
    main()
```

This lets you put behaviour in a file that will only be executed if it's run
from the command line, not when it's included in a program.

So it's possible to achieve the same thing in bash, simply using:

```bash
[[ $0 != -${SHELL##*/} ]] && main
```

Could that be clearer? Probably. Here's what it means from left to right:

* `[[` are extended conditional tests. Useful if you want to use non-posix
operators
such as the regex matcher (`=~`), but in this case the posix conditional
`[` is directly substitutable.
* `$0` is the zeroth command line argument. In case you've just called
`./script.bash` It's going to be, you guessed it, `script.bash`. When we
source the same file(`source script.bash`) the zeroth argument contains an
interesting value which will be revealed in a moment.
* `!=` Not equal, right?
* `-${SHELL##*/}` So that *interesting value* I mentioned when a script is
sourced is the
name of the shell preceded by a hyphen. If you're running a bash shell, that
will be the value `-bash`. the `$SHELL` environment variable contains the
path of the current shell, for example `/bin/bash`. The matcher `##*/` removes
the longest string (`##`) of any characters (`*`) up to a `/` from the variable
`SHELL`. In effect this is doing the same thing as `basename`, but without
spawning a subshell.
* `&& main` is just shorthand for `if [ true ]; then main; fi`, and it means
the main function will be called if the condition is true.

If you try this on the command line, you'll see that executing the script
directly evaluates the body of main, while sourcing does not.

This opens up the possibility to write shared code in executable scripts,
and to separate unit tests out from the runnable code. My goal is to write
clean, extensible, modular bash.

### Caveats

* I haven't tested this with many non-bash shells. If you're the kind of
devient who uses zsh, korn, or an even less savoury offering, this may not
work for you.
* I have used this successfully running bash 3, 4 and on busybox using the
ash shell.
* I have a hazy understanding of `source` so I can imagine a few failure cases
in multi-shell environments where the primary shell is not bash. If anyone can
suggest a better solution, do contact me.
