---
title: "Loading a DTD from a JAR"
date: 2013-12-02
---

Bundling resources into a jar file is a common way to deploy your Java
application, useful because it makes the jar's resources available to
retrieve via the classloader at runtime.

When loading XML, you can require DTD veirification, which is a means of
validating the XML is gramatical. Hopefully everything up till now is old
news.

I struggled trying to load XML from the classpath to also have the DTD load
correctly. In the case of my old application the path to the DTD is specified
in the XML as a relative path, and it's assumed the document is on the
filesystem.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE MyApplication SYSTEM "dtd/config.dtd">
<config>
...
</config>
```

Following some redesign, this is no longer the case. So how do we find
`config.dtd` now that it's on the classpath? It's really simple once you know
which components to use; the answers are [here on Stack Overflow](http://stackoverflow.com/a/155330),
and in more depth [here at onjava.com](http://www.onjava.com/pub/a/onjava/excerpt/java_xslt_ch5/index.html?page=5).

Applying this solution, I wrote a custom entity resolver which transforms
`dtd/config.dtd` into the classpath resource path `/dtd/config.dtd`. For
all other resources it does nothing, which is what I want.

```java
SAXBuilder builder = new SAXBuilder(true);
    builder.setEntityResolver(new EntityResolver() {
    @Override
    public InputSource resolveEntity(String publicId, String systemId) throws SAXException, IOException {
        if (systemId.endsWith("dtd/config.dtd")) {
            return new InputSource(getClass().getResourceAsStream("/dtd/config.dtd"));
        }
        return null;
    }
});
```

If you haven't come across entity resolvers, this problem seems a bit daunting,
but it's much simpler than it first appears.
