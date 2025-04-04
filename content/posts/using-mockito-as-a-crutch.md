+++
date = '2013-08-28'
title = 'Using Mockito as a Crutch'
+++

*"omne trium perfectum"*, as the Latins used to say: a set of three is perfect. Sadly I don't have three facts I know about [mockito](http://code.google.com/p/mockito/),
but I do have two facts. That's not enough to impress anyone, which means
almost by definition, it's enough for a blog.

### Returning different values from a mock on successive calls

This is so basic, I expect I'm the last person to hear about it.
I stumbled on it while trying to mock an [`Enumeration`](http://docs.oracle.com/javase/6/docs/api/java/util/Enumeration.html). The `thenReturn`
method takes an array of objects which will be returned over subsequent
calls, for example:

{% highlight java %}
Enumeration myEnumeration = mock(Enumeration.class);
when(myEnumeration.hasMoreElements()).thenReturn(true, true, true, true, false);
when(myEnumeration.nextElement()).thenReturn(new Integer(3), new Integer(5));

// ...

while(myEnumeration.hasMoreElements()) {
    myEnumeration.nextElement(); // returns 3, 5, 3, 5
}
{% endhighlight %}

The return value will wrap over the given parameters, so the third call to
`nextElement` will return the zeroth parameter and so on. This is a really
clean way of mocking objects with simple, predictable behaviour.

### Integrating an existing object with complex behaviour

In the code I'm working on, our configuration is passed around as a
`properties` object. It's pretty clear that every time a developer has
been called upon to support some new configuration, they've created a
wrapper object somewhere in the code base close to where it's needed.
This is kind of horrible for three reasons:

1. the same configuration is used in multiple places, so we end up with
multiple instances of some of these objects;

2. supporting a new configuration format means duplicating lots of
logic that determines how to access the new config, and when to
fall back to the old config

3. THE DEPENDENCIES, OH GOD, THE DEPENDENCIES!

I went with creating a composite of old and new configuration, and
where behaviour contained in supporting objects was too complex to
refactor in a reasonable amount of time, I wrapped it with the composite
configuration object to hide it from the rest of the system.

Mockito's `thenAnswer` method lets you model arbitrary behaviour
in response to a method call on a mock. This example passes an argument
to an existing class.

{% highlight java %}
NewConfiguration config = mock(NewConfiguration.class);  // mock new class
when(config.doComplexThing(anyString())).thenAsnwer(new Answer() {
    Object answer(InvocationOnMock invocation) {
        new ExistingObject().doComplexThing(invocation.getArguments()[0]));  // call existing method
    }
});
{% endhighlight %}

I'm not advocating these techniques for doing clean TDD,  but they're good
tools for cleaning up existing code.
