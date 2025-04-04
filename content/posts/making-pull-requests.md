+++
categories = ['development']
date = '2014-05-24'
description = 'I just got my first pull request accepted!'
lastmod = '2014-05-24'
title = 'Making Pull Requests'
+++

![Nyan Cat](https://octodex.github.com/images/nyantocat.gif)

Excited much? Yes I am. I have wanted to contribute to an open source project
for a really, really long time. I am unreasonably psyched about the *whole*
experience right now.

**Trigger warning**: I am going to be unironically enthusiastic. Sorry.

## Here is how I did the do ##

It was unbelievably easy. I noticed in the github news stream that one of
my friends had starred a project, and it looked kind of cool. The project
was [`micahflee/onionshare`](https://github.com/micahflee/onionshare) and I
liked it because I like [Tor](https://www.torproject.org/) and I think
onionshare is awesome and I can absolutely see someone needing to use it.
By the way, I can't overemphasize how important Tor is. You can totally
[help out](https://cloud.torproject.org/) even if you just have some extra
[cash to throw at it](https://www.torproject.org/donate/donate.html.en).

So I looked at the open issues. At the time there were two similar issues, one
about [Localization](https://github.com/micahflee/onionshare/issues/8) and
a pull request to [Add Spanish translations](https://github.com/micahflee/onionshare/pull/13).
That's when I realised "I can read in other languages, okay, not at all well,
but I can definately do it!" So I cloned `micahflee/onionshare`. The code
was very nice, I had no trouble figuring out what I would have to do, I could
see that the i18n strings were loaded from a json file.

I spent the next about an hour researching what I hoped were good translations
by typing what I thought was the correct phrasing into google.fr, and checking
how well it matched what French speakers actually use on message boards, in
blogs and so on. It was a bit of a challenge to match the style of language in
the English translation, and I think I sucked pretty hard.

I forked `micahflee/onionshare` to create my own repo, and cloned it. On my
machine I edited the `strings.json` file with my lovingly collected French
words, staged, committed and pushed.

Then I got stage fright. My translations are all wrong. The patch is totally
trivial. I'm not even French. But I really wanted to contribute something,
and that pushed me to create the pull request. It sounds like a little thing,
but I suppose it's a bit like being new at school, it feels like there is a
lot of pressure to make a good impression. It *is* a little thing, but it was
actually the hardest part.

It took maybe an hour before my pull request was merged, and it was pretty
awesome. Unlike when I commit code at work,
[I got this](https://github.com/micahflee/onionshare/pull/14#issuecomment-44108128):

> Thanks so much.

Somebody immediately gave a shit. That felt good.

## Why so long? ##

I've been a software engineer for two years, and it's taken me all this time to
make a contribution. I think gaining the ability to contribute has been
a combination of confidence that I *have* something to contribute, finding a
project I strongly identify with, and a bunch of great blogs and other sources
of engouragement online...

### Here are a bunch of great blogs and resources ###

[How to be an open source gardener](http://words.steveklabnik.com/how-to-be-an-open-source-gardener)

The big takeaway from this was that you don't need to be a huge personality to
start making contributions to open source. This was a *major* motivator.

[codefirefox.com](http://codefirefox.com/)

Watching the codefirefox videos made me think "I could totally contribute to
firefox", and if I could do that I could do anything. I could be like superman
(⌥). But with typing. The videos are a really good introduction to some common
open source processes with what I guess is just common sense advice.

[@b0rk](https://twitter.com/b0rk)

Julia Evans is just infectiously enthusiastic about programming. I both read
the blog and watched the video about her writing an operating system in [rust](http://www.rust-lang.org/)
and it made me want to get involved with awesome projects and people. There
are a bunch of people on Twitter who are extremely motivational, I'm sure
I could compile a list, but the rust talk was kind of a big deal.

## Why do? ##

Here are the positives from my tiny little open source experience and why I
would do it again:

* I'm getting brand new skills, I am a Level 1 Open Source Contributor
(chaotic good)
* I feel unreasonably good about helping out
* It made me nervous, but I did it anyway and that felt like a little win
* Github is gaming me (with contribution counts and buttons to click and
oh the commit chart) I CAN BEAT YOU GITHUB!
* There is a lot less pressure on me now. The 'look at some issues, say hi to
some dudes, submit a patch' hoopla is no longer completely foreign

I'm sure there are lots of other good reasons too. There were no
downsides.

It was awesome!
