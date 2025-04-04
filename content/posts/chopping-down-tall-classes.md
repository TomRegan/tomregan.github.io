+++
date = '2013-08-12'
title = 'Chopping Down Tall Classes'
+++

I've been having bags of fun today trying to get to grips with an
enormously busy class. The carbuncle of code is basically a
data structure that contains a tree of values loaded from a
plain-text configuration file. To keep it from being too simple and
boring, somebody decided you should be able to pass it an instance
of our application, and have it call lots of configuration methods
on it for you.

<img src="http://www.tumblr.com/photo/1280/explodingdog/1359175167/1/tumblr_lalh5m7hw51qzs63f"/ style="width: 50%"/>

What I actually want it to do is to expose a public API, so we can
let our customers, you know, use it to configure their application. The
quickest way I reckon on doing this is through [interface segregation](https://en.wikipedia.org/wiki/Interface_segregation_principle).

<img src="http://www.explodingdog.com/drawing/myrobotgarden.gif" style="width: 50%" />

My public interface is a small bunch of accessors that return
the parsed configuration values. The existing class is extended
to implement my interface, so I can return an instance of it
without users having to understand all of its quirky behaviour.

{% highlight java %}
public interface UserFriendlyConfiguration {
    /**
     * That thing you're so interested in. This is that thing.
     * @param option  That option you want.
     * @return That thing you're interested in.
     */
    public String getInterestingValue(String option);
}

public class BigConfigurationClass implements UserFriendlyConfiguration {
    private StuffBag<Stuff> stuff;
    public void configureAllTheThings(DistantRelation ohaiNotExpectinYou) {
        // good luck decoupling this
    }

    // lines and lines and lines

    @Override
    public String getInterestingValue(String option) {
        return stuff.get(option);
    }
}

public class Application {
    private BigConfigurationClass configuration;
    public UserFriendlyConfiguration getConfiguration() {
        return configuration;
    }
}
{% endhighlight %}

*And yes, my documentation* really *is that good.*

I gave some thought to creating a wrapper class to achieve more or less the
same thing, but I like the minimalism of the interface approach. It's
pretty compelling because it's about the least complex way of changing
the code while still meeting our goals, and it means I can spend more time
chopping the über class down into single responsibility objects, for
which I'm sure someone will someday thank me.
