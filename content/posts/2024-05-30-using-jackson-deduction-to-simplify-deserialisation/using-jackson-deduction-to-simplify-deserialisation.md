+++
date = '2024-05-30'
tags = ['java', 'jackson']
title = 'Using Jackson Deduction to Simplify Deserialisation'

[cover]
  image = 'posts/2024-05-30-using-jackson-deduction-to-simplify-deserialisation/cover.jpeg'
+++

I find the documentation for Jackson is on the terse side, and dare I
say obstructively self-referential. If you aren't moved fully to tears
by the Javadoc entries for `JsonTypeInfo.As`, you deserve an ACM award.

In short: I need a helpful and up-to-date guide, so I'm writing one.

# What are we trying to do? {#_what_are_we_trying_to_do}

We often want to vary the content of our JSON. It's integral to the
[Envelope pattern](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EnvelopeWrapper.html)
for instance. Varying the content looks like this.

Here's an example of an intergalactic animal.

``` json
{
  "created": "1977-05-25T12:00:00Z",
  "animal": {
    "galaxy": "Far Far Away",
    "name": "Womp Rat"
  }
}
```

And here's an example of a provincial animal.

``` json
{
  "created": "1835-09-15T06:00:00Z",
  "animal": {
    "province": "Galápagos Islands",
    "name": "Galápagos tortoise"
  }
}
```

In our Java code, we want to be able to deserialise Animals, and we want
to handle either `IntergalacticAnimals`, or `ProvincialAnimals`. We
might write some transport code that looks like this.

``` java
record AnimalSpottedEvent<A extends Animal>(Instant created, A animal) {}

interface Animal {}

record IntergalacticAnimal(String galaxy, String name) implements Animal {}

record ProvincialAnimal(String province, String name) implements Animal {}
```

When it's called to handle deserialisation of an Animal over the wire,
Jackson needs to work out what's the \"message inside the envelope?\" Is
it an `IntergalacticAnimal`, or is it a `ProvincialAnimal`? Without this
knowledge, Jackson can't instantiate the right kind of object.

There is a (truly excellent) resource, [Deserialize JSON with Jackson
into Polymorphic Types - A Complete
Example](https://programmerbruce.blogspot.com/2011/05/deserialize-json-with-jackson-into.html),
which describes how to do this using `JsonTypeInfo`, but it was written
in 2011 for Jackson 1.7.

# State of the Art {#_state_of_the_art}

You may have missed the 2.12.0 release of Jackson Databind in Novemeber
2020. The world was generally busy with other things around then. In
which case the
[`JsonTypeInfo.Id#DEDUCTION`](https://fasterxml.github.io/jackson-annotations/javadoc/2.12/com/fasterxml/jackson/annotation/JsonTypeInfo.Id.html#DEDUCTION)
could have slipped by you unnoticed.

The newer \"deduction\" mode takes away the need to tell Jackson *how*
to read the incoming data; instead Jackson will take a list of possible
choices you give it, and it will determine the best match for
deserialisation.

What does that look like? You can follow these steps:

1.  add `@JsonTypeInfo(use = DEDUCTION)` to the envelope property

2.  add `@JsonSubTypes(@Type(...))`

Here is our updated example code.

``` java
record AnimalSpottedEvent<A extends Animal>(
        Instant created,
        @JsonTypeInfo(use = DEDUCTION)
        @JsonSubTypes({
                @Type(IntergalacticAnimal.class),
                @Type(ProvincialAnimal.class)})
        A animal) {}

interface Animal {}

record IntergalacticAnimal(String galaxy, String name) implements Animal {}

record ProvincialAnimal(String province, String name) implements Animal {}
```

[I've created a demo
project](https://github.com/TomRegan/jackson-deduction-demo) as a
companion to provide a working example.

Happy deserialisation!

# Appendix {#_appendix}

## Dependencies {#_dependencies}

`jackson-annotations` provides `@JsonTypeInfo` and associated
annotations which you'll use in your transport code.

`jackson-databind` provides the \"invisible\" deserialisers. You'll need
these available on the classpath in your server or client application.

There are any number of ways to get `jackson-annotations` and
`jackson-databind` on your classpath. `spring-boot-starter-web` includes
many Jackson components; alternatively the Jackson project provides a
[BOM](https://mvnrepository.com/artifact/com.fasterxml.jackson/jackson-bom).
