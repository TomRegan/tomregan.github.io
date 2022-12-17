---
title: Fixing a Permissions Error Opening a Semaphore
date: 2012-10-06
---

Today I spent a couple of hours getting semaphores to work.

![](https://farm1.staticflickr.com/177/403268783_bd644e1b0a.jpg "semaphore by brapps, on Flickr")

I was opening a semaphore like this:

{% highlight c %}
if (SEM_FAILED == (CACHE_LOCK = sem_open("/cache_lock", O_CREAT, 0660, 1))) {
    printf("error opening semphore: %i\n", errno);
    return -1; /* error creating semaphore */
}
{% endhighlight %}

This runs fine on my Mac, but fails on an Ubuntu machine. On failure it sets
`errno` to `EACCESS (13)`. Whatever you might expect, this fact alone did
not allow me to figure out the cause of the problem.

![](/assets/images/2012-10-06-fixing-a-permissions-error-opening-a-semaphore/read-all-the-things.png)

It turns out the name I'd chosen isn't allowed. I couldn't find the rules
that expressly state this, or even hinted that it might be the case, but
I found when I changed the name from '/cache_lock' to '/lock' on a whim,
it worked. Apparantly, underscores are not cool with Linux kernel developers.

During my Googling I found [the following question](http://stackoverflow.com/questions/10409901/sem-open-causes-illegal-seek-error)
which put me on the right track.

