---
title: Switching Text Editors
date: 2013-09-08
---

After 7 years, I'm finally doing the dirty dead: jumping off the
Vim ship, into the [*adjective*], [*adjective*] something [*noun*]
that is emacs. There follows a brief description of the process so far.

![Gnu by Dean Gugler at flickr CC BY-NC]
(https://farm7.staticflickr.com/6173/6169267550_753ebd220b.jpg)

<em>
<small>
<a href="https://secure.flickr.com/photos/67335469@N00/6169267550/in/photolist-apa7iU-c2h7bf-8dNaw4-9YRiUm-9YNqoc-k92NB-c2eXqw-Ahtjx-3WtN1f-uGgQz-8V1EY6-4JZqBQ-4mQasz-4mtRjz-5rnwNR-d8zxLo-64pxUe-pZvNx-mXKrB-6gmUKL-6o3mJG-jsC8d-5pxSjD-9YS6ur-9YS8de-9YS6ik-5dyNy2-5dD91Q-5dD8WU-5dD9d5-5pxTdp-9YNrDc-9YRjtw-7o6xWC-7o6xXC-9YRkJN-9YRk5J-8PszGo-avTotn-9YNpeM-9YNqWe-fcymYw-fcysNy-6gcEeC-5ugFB8-8Sv3V6-8SyDkf-8Uxpak-8UAs4b-8UArQh-8UArwG">Gnu</a>
by
<a href="https://secure.flickr.com/photos/ontario_wanderer/">Dean Gugler</a>
at flickr
(<a href="https://creativecommons.org/licenses/by-nc/3.0/">CC BY-NC</a>)
</small>
</em>

### Install a recent version of emacs

My version of OS X has emacs 22.2 installed, which is way out of
date. To install the most recent version, I used homebrew.

So, first step, `brew install emacs`. There are a few options (which
you could see with `brew options emacs`) that make it possible to build
a cocoa version amongst other things.

### emacs' little MELPA

One of my aims was to create a configuration from scratch that I could
understand, and that matched the functionality of my Vim set-up.

I created a fresh `init.el` throwing out the old `.emacs.d` directory I'd
been experimenting with, and added the MELPA repository as described
[here](https://github.com/milkypostman/melpa). The standard repository is
a bit of a bare cupboard compared with MELPA. The configuration
to add a repo is a simple few lines of elisp which could go in an
`init.el` file:

{% highlight lisp %}
(require 'package)
 (add-to-list 'package-archives
   '("melpa" . "http://melpa.milkbox.net/packages/") t)
 (package-initialize)
{% endhighlight %}

then restart emacs and `M-x package-list-packages`. I poked around for a
while and came up with a short list of packages I thought might be useful.

### elisp-p?

I haven't written anything useful using lisp up to now, which was actually
one of the attractions of using emacs. I plan to use emacs
on a couple of machines, and I want to be able to check my [configuration
repository](https://github.com/TomRegan/home-service)
out of github, start emacs and have all my packages install
automagically. The following achieves this:

{% highlight lisp %}
(defvar packages
  '(busybee-theme
    git-commit-mode
    git-rebase-mode
    google-c-style
    ido-ubiquitous
    markdown-mode
    markdown-mode+
    rainbow-mode
    undo-tree
))

(defun packages-installed-p (packages)
  (if (not packages) t
    (if (package-installed-p (car packages)) 
    (packages-installed-p (cdr packages))
      nil)))

(defun install-packages (packages)
  (mapc 'install-package packages))

(defun install-package (package)
  (unless (package-installed-p package)
    (package-install package)))

(defun install-missing-packages ()
  (unless (packages-installed-p packages)
    (package-refresh-contents)
    (install-packages packages)))

(install-missing-packages)
{% endhighlight %}	

When packages are missing, they are installed; there's no automatic cleanup,
so installed packages which aren't listed are ignored rather than deleted.

The remainder of my installation is arranged like this:

    .emacs.d/
	|- custom/
	|  |- packages.el  ;; manages packages
	|  `- editor.el    ;; contains editor settings
	|- elpa/
	`- init.el         ;; sources files from custom/

I'm working at the moment to configure a modern environment for large C
projects, which will doubtless lead to another tedious post in the
near future.

My emacs directory, which might serve as some kind of modest example to the
completely and utterly uninitiated, is [here](https://github.com/TomRegan/home-service/tree/develop/emacs.d).
