+++
date = '2013-08-18'
title = 'Hanging Out with Linked Hash Maps'
+++

[`LinkedHashMap`](http://docs.oracle.com/javase/6/docs/api/java/util/LinkedHashMap.html)
is a surprising useful data structure, and missing from many standard
libraries.

In my case I have class that represents a message and contains a set of
headers. Imagining an HTTP message might be useful. Historically these sets
have been mapped using a configuration file that dictates the order of field
keys and values (primarily it's done this way to support ordered value-only
fields):

<pre>
0    key 1
1    value 1
2    key 2
...
</pre>

So, to support messages for which the index has no special significance,
we're basically implementing a dictionary inside a list. The
`LinkedHashMap` has a few features I don't need, for instance I don't
care about access time so I don't particularly need a hash map (but it
gives me a set cheaply), and I don't need to iterate in both directions
(but I do want to pull the fields out in insertion order).

The `LinkedHashMap` like `HashMap` is a collection of buckets. Duplicates
aren't permitted so we get set behaviour, and there's a performance benefit
for look-ups. Additionally, `LinkedHashMap` maintains a bi-directional link
between elements, so iterators return elements in insertion order and
can be navigated forwards and backwards.

There's a moderately helpful SO answer to [this question](http://stackoverflow.com/questions/2889777/difference-between-hashmap-linkedhashmap-and-sortedmap-in-java) that gives a bit of detail on the different children of `Map`. My Java
adventure continues....
