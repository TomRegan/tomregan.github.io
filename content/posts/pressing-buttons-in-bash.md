+++
date = '2013-05-12'
title = 'Pressing Buttons in Bash'
+++

Shortcuts are a source of constant amusement as far as I'm concerned, and
nothing on Earth makes me more ecstatically happy than accidentally
discovering a new one. Today's mis-fingering was `^t` in bash, which
roughly translated means *transpose the character under the cursor and
the previous character*.

Two transposed characters is by far my commonest typo (vim: `xp`), so
I'm not going to lie, I'm pretty happy. To celebrate I'm going to try and
use bash this week by only touching letter keys, `Ctrl`, `Alt` and `Shift`.
Here's how:

Basic Movement:

- `^f` cursor right
- `^b` cursor left
- `^m` enter
- `^i` tab
- `^h` backspace
- `^a` start of line
- `^e` end of line
- `^u` yank back
- `^k` yank forward
- `^w` yank word back
- `^y` paste

Awesome sauce:

- `^p` previous in history
- `^n` next in history
- `^r` reverse incremental search
- `^g` clear search
- `^l` clear screen

In OS X's `Terminal.app`,  to enable `Alt` key combinations,
select 'Use option as meta key' in its keyboard preferences.
