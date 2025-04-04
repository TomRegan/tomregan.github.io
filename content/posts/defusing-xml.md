+++
date = '2013-04-15'
title = 'Defusing XML'
+++

![](https://gs1.wac.edgecastcdn.net/8019B6/data.tumblr.com/tumblr_m8uhnbZZoM1rzupqxo1_500.png)

*XML: the most terrifying thing on the planet*

Rule 1 of the Internet&mdash; is never, ever trust the Internet. Reading from a
socket is the equivalent of putting your hand into an unknown box that's full
of snakes, but you don't know it's full of snakes and it ends up pretty
touch-and-go as to whether you get out in one piece.

Getting XML over the wire is like that, but *more* terrifying and *more*
dangerous.

Here's one heinous example:

### Exponential Entity Expansion

By defining entities which form chains that fan out exponentially when they're
dereferenced, the load and memory used on a host can be attacked.
[It goes a little something like this](https://www.youtube.com/watch?v=s0EcAJZpoZ4):

{% highlight xml %}
<!DOCTYPE xmlbomb [
<!ENTITY a "1234567890" >
<!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;">
<!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;">
<!ENTITY d "&c;&c;&c;&c;&c;&c;&c;&c;">
]>
<bomb>&d;</bomb>
{% endhighlight %}

Each reference triggers a growing number of dereferences, turning the few
hundred bytes above into a multi gigabyte ordeal.

Exponential entity expansion is about as much fun as a wet fart.

### External Entity Expansion

More fun along similar lines: using the `SYSTEM` identifier, an external
resource can be loaded and, if you've very lucky, end up in a bit of arbitrary
code execution.

> When the URI is a URL (e.g. a http:// locator) some parsers download the
> resource from the remote location and embed them into the XML document
> verbatim. (Explains [Christian Heimes](https://pypi.python.org/pypi/defusedxml/).)

Maybe I'll start giving these ratings. That one gets 4 / 5 frowny faces.

### DTD Retrieval

Nearly identical to External Entity Expansion. But for DTDs. There's a
further variant for local files too, if you aren't depressed enough.

### PANIC!
Thankfully there are lovely libraries (*mmmmmm* libraries) for python.
`defusedxml` is now available on pypy and there are some good best
practices to follow in [this python.org blog post](http://blog.python.org/2013/02/announcing-defusedxml-fixes-for-xml.html).

Also worth a read are the following:

* [Pypy listing for defusedxml](https://pypi.python.org/pypi/defusedxml/)
* [Attacking XML Security Message Oriented Madness, XML Worms and Web Service Security Sanity, by Brad Hill](https://www.isecpartners.com/media/12976/iSEC-HILL-Attacking-XML-Security-bh07.pdf) (slides, PDF)
* [XML Denial of Service Attacks and Defenses, MSDN](http://msdn.microsoft.com/en-us/magazine/ee335713.aspx)
